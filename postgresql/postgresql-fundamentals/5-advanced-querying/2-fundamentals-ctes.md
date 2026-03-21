---
source_course: "postgresql-fundamentals"
source_lesson: "postgresql-fundamentals-ctes"
---

# Common Table Expressions (CTEs)

## Introduction
CTEs make complex queries readable by breaking them into named, reusable steps using the WITH clause. Instead of nesting subqueries inside subqueries, you define each step as a named temporary result set. CTEs also support recursion, enabling queries on hierarchical data like org charts and threaded comments.

## Key Concepts
- **CTE (Common Table Expression)**: A named temporary result set defined with the WITH clause. It exists only for the duration of the query.
- **WITH clause**: The keyword that introduces one or more CTEs before the main SELECT.
- **Recursive CTE**: A CTE that references itself, used for hierarchical or tree-structured data.
- **Materialization**: PostgreSQL may materialize (compute and store) CTE results, which can be beneficial or detrimental depending on the query.

## Real World Context
CTEs are used constantly in data analysis and reporting. Instead of writing a 50-line nested query, you break it into 3-4 named CTEs that each do one clear thing. Recursive CTEs solve problems that are nearly impossible with regular SQL: traversing org charts, bill of materials, category trees, and threaded discussions.

## Deep Dive

### Basic CTE Syntax

A CTE is defined with WITH and used like a table in the main query:

```sql
WITH cte_name AS (
    SELECT ...
)
SELECT * FROM cte_name;
```

The CTE is like a temporary view that exists only for this query.

### Simple Example

Break a complex query into readable steps:

```sql
-- Find books by prolific authors (3+ books)
WITH prolific_authors AS (
    SELECT author_id, COUNT(*) AS book_count
    FROM books
    GROUP BY author_id
    HAVING COUNT(*) >= 3
)
SELECT 
    b.title,
    a.name,
    pa.book_count
FROM books b
JOIN prolific_authors pa ON b.author_id = pa.author_id
JOIN authors a ON b.author_id = a.id;
```

The CTE identifies prolific authors first, then the main query joins against it. This is much clearer than a nested subquery.

### Multiple CTEs

You can define multiple CTEs, each building on the previous:

```sql
WITH 
    monthly_sales AS (
        SELECT 
            DATE_TRUNC('month', created_at) AS month,
            SUM(total) AS revenue
        FROM orders
        GROUP BY DATE_TRUNC('month', created_at)
    ),
    average_sales AS (
        SELECT AVG(revenue) AS avg_monthly
        FROM monthly_sales
    )
SELECT 
    ms.month,
    ms.revenue,
    CASE 
        WHEN ms.revenue > avs.avg_monthly THEN 'Above Average'
        ELSE 'Below Average'
    END AS performance
FROM monthly_sales ms
CROSS JOIN average_sales avs
ORDER BY ms.month;
```

The second CTE (`average_sales`) references the first CTE (`monthly_sales`), creating a clean pipeline.

### Recursive CTEs

Recursive CTEs reference themselves and are essential for hierarchical data:

```sql
-- Employee hierarchy
WITH RECURSIVE employee_tree AS (
    -- Base case: top-level employees
    SELECT id, name, manager_id, 1 AS level
    FROM employees
    WHERE manager_id IS NULL
    
    UNION ALL
    
    -- Recursive case: employees under managers
    SELECT e.id, e.name, e.manager_id, et.level + 1
    FROM employees e
    JOIN employee_tree et ON e.manager_id = et.id
)
SELECT * FROM employee_tree ORDER BY level, name;
```

The base case selects root nodes (employees with no manager), and the recursive case finds their direct reports, repeating until no more rows are found.

### CTE vs Subquery

Here is when to use each approach:

| Feature | CTE | Subquery |
|---------|-----|----------|
| Readability | Better for complex queries | Better for simple cases |
| Reusability | Can reference multiple times | Must repeat |
| Recursion | Supported | Not supported |
| Optimization | Materialized in some cases | Always inlined |

A CTE can be referenced multiple times in the same query, which is impossible with a subquery:

```sql
WITH stats AS (
    SELECT author_id, COUNT(*) AS books, AVG(pages) AS avg_pages
    FROM books GROUP BY author_id
)
SELECT * FROM stats WHERE books > (SELECT AVG(books) FROM stats);
```

Here `stats` is used twice — once in the main query and once in the WHERE subquery.

## Common Pitfalls
1. **Over-using CTEs for simple queries** — A simple subquery is clearer than a CTE for one-off, non-reused expressions. Use CTEs when the query has multiple steps or the result is referenced more than once.
2. **Forgetting RECURSIVE keyword** — Self-referencing CTEs require `WITH RECURSIVE`. Without it, PostgreSQL throws an error.

## Best Practices
1. **Name CTEs descriptively** — `monthly_sales` is much clearer than `t1` or `cte1`. The name should describe what the CTE contains.
2. **Use CTEs to replace deeply nested subqueries** — If you have more than two levels of nesting, rewrite with CTEs.
3. **Add a termination condition for recursive CTEs** — Always ensure recursive CTEs will terminate, or add a LIMIT or depth check.

## Summary
- CTEs are named temporary result sets defined with WITH, making complex queries readable.
- Multiple CTEs can be chained, each building on the previous.
- Recursive CTEs handle hierarchical data (org charts, trees, threaded comments).
- CTEs can be referenced multiple times in the same query, unlike subqueries.
- Use descriptive names for CTEs and add termination conditions for recursive ones.

## Code Examples

**A recursive CTE that generates all dates in January 2024 — useful for reports that need every day, even those with no data**

```sql
-- Recursive CTE: generate a series of dates
WITH RECURSIVE date_series AS (
    SELECT DATE '2024-01-01' AS day
    UNION ALL
    SELECT day + INTERVAL '1 day'
    FROM date_series
    WHERE day < DATE '2024-01-31'
)
SELECT day FROM date_series;
```


## Resources

- [WITH Queries (CTEs)](https://www.postgresql.org/docs/18/queries-with.html) — Complete guide to Common Table Expressions

---

> 📘 *This lesson is part of the [PostgreSQL Fundamentals](https://stanza.dev/courses/postgresql-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*