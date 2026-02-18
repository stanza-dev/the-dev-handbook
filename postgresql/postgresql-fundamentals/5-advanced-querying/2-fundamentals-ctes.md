---
source_course: "postgresql-fundamentals"
source_lesson: "postgresql-fundamentals-ctes"
---

# Common Table Expressions (CTEs)

CTEs make complex queries readable by breaking them into named steps.

## Basic CTE Syntax

```sql
WITH cte_name AS (
    -- Your query here
    SELECT ...
)
SELECT * FROM cte_name;
```

## Simple Example

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

## Multiple CTEs

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

## Recursive CTEs

For hierarchical data (org charts, categories, threads):

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

## CTE vs Subquery

| Feature | CTE | Subquery |
|---------|-----|----------|
| Readability | Better for complex queries | Better for simple cases |
| Reusability | Can reference multiple times | Must repeat |
| Recursion | Supported | Not supported |
| Optimization | Materialized in some cases | Always inlined |

```sql
-- CTE referenced twice
WITH stats AS (
    SELECT author_id, COUNT(*) AS books, AVG(pages) AS avg_pages
    FROM books GROUP BY author_id
)
SELECT * FROM stats WHERE books > (SELECT AVG(books) FROM stats);
```

📖 [WITH Queries](https://www.postgresql.org/docs/18/queries-with.html)

## Resources

- [WITH Queries (CTEs)](https://www.postgresql.org/docs/18/queries-with.html) — Complete guide to Common Table Expressions

---

> 📘 *This lesson is part of the [PostgreSQL Fundamentals](https://stanza.dev/courses/postgresql-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*