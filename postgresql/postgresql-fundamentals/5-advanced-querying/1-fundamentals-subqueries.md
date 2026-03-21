---
source_course: "postgresql-fundamentals"
source_lesson: "postgresql-fundamentals-subqueries"
---

# Subqueries

## Introduction
A subquery is a query nested inside another query. They help solve complex problems step by step — you can use the result of one query as input to another. Subqueries are one of the most powerful tools in SQL.

## Key Concepts
- **Scalar subquery**: A subquery that returns a single value (one row, one column). Used with comparison operators like `=`, `>`, `<`.
- **List subquery**: A subquery that returns a list of values (multiple rows, one column). Used with `IN`, `NOT IN`, `ANY`, `ALL`.
- **Correlated subquery**: A subquery that references columns from the outer query. It runs once per row of the outer query.
- **EXISTS**: Tests whether a subquery returns any rows. More efficient than IN for large datasets.
- **Derived table**: A subquery in the FROM clause that acts as a temporary table.

## Real World Context
Subqueries solve problems that cannot be expressed in a single flat query: "find users who ordered more than the average", "find products that have never been ordered", "show each employee's salary relative to their department's average". These patterns appear in every real application.

## Deep Dive

### Scalar Subqueries

Scalar subqueries return a single value and can be used anywhere a single value is expected:

```sql
-- Books longer than average
SELECT title, pages
FROM books
WHERE pages > (SELECT AVG(pages) FROM books);

-- Latest order
SELECT *
FROM orders
WHERE created_at = (SELECT MAX(created_at) FROM orders);
```

The inner query computes one value (average pages, maximum date), and the outer query uses it as a filter.

### Subqueries with IN

IN subqueries return a list of values to filter against:

```sql
-- Books by authors from the USA
SELECT title
FROM books
WHERE author_id IN (
    SELECT id 
    FROM authors 
    WHERE country = 'USA'
);

-- Users who have never ordered
SELECT name
FROM users
WHERE id NOT IN (
    SELECT DISTINCT user_id 
    FROM orders
);
```

The inner query produces a list of IDs, and the outer query checks membership in that list.

### Correlated Subqueries

Correlated subqueries reference the outer query, running once per row:

```sql
-- Authors with more than 3 books
SELECT name
FROM authors a
WHERE (
    SELECT COUNT(*) 
    FROM books b 
    WHERE b.author_id = a.id
) > 3;
```

Notice `b.author_id = a.id` — this references the outer query's `a.id`, making it correlated. The subquery runs for each author.

### EXISTS Subqueries

EXISTS checks whether any rows match, and is often more efficient than IN:

```sql
-- Authors who have written at least one book
SELECT name
FROM authors a
WHERE EXISTS (
    SELECT 1 
    FROM books b 
    WHERE b.author_id = a.id
);

-- Products never ordered
SELECT name
FROM products p
WHERE NOT EXISTS (
    SELECT 1 
    FROM order_items oi 
    WHERE oi.product_id = p.id
);
```

EXISTS stops as soon as it finds one matching row, making it efficient for large tables.

### Subqueries in FROM (Derived Tables)

Subqueries in the FROM clause create temporary tables:

```sql
-- Average of category averages
SELECT AVG(category_avg) AS overall_avg
FROM (
    SELECT category, AVG(price) AS category_avg
    FROM products
    GROUP BY category
) AS category_averages;
```

The inner query groups by category; the outer query aggregates those results. For complex cases, CTEs (next lesson) are often clearer.

## Common Pitfalls
1. **NOT IN with NULLs** — If the subquery returns any NULL values, `NOT IN` returns zero rows because NULL comparisons are always unknown. Use `NOT EXISTS` instead for safety.
2. **Correlated subqueries and performance** — Correlated subqueries run once per row in the outer query, which can be very slow on large tables. Consider rewriting with JOINs or CTEs.

## Best Practices
1. **Prefer EXISTS over IN for large datasets** — EXISTS stops at the first match, while IN must evaluate the entire list.
2. **Use NOT EXISTS instead of NOT IN** — NOT EXISTS handles NULLs correctly, while NOT IN does not.
3. **Consider CTEs for complex subqueries** — When subqueries get nested or repeated, CTEs (WITH clause) improve readability.

## Summary
- Scalar subqueries return one value; list subqueries return multiple values.
- Correlated subqueries reference the outer query and run once per row.
- EXISTS is more efficient than IN for checking existence in large tables.
- NOT EXISTS is safer than NOT IN because it handles NULLs correctly.
- Derived tables (subqueries in FROM) create temporary tables for further querying.
- For complex cases, prefer CTEs over deeply nested subqueries.

## Code Examples

**A correlated subquery that compares each product's price to the average price of its own category**

```sql
-- Find products priced above their category's average
SELECT p.name, p.price, p.category
FROM products p
WHERE p.price > (
    SELECT AVG(p2.price)
    FROM products p2
    WHERE p2.category = p.category  -- Correlated!
)
ORDER BY p.category, p.price DESC;
```


## Resources

- [Subquery Expressions](https://www.postgresql.org/docs/18/functions-subquery.html) — Official documentation on subquery expressions including EXISTS, IN, and ANY

---

> 📘 *This lesson is part of the [PostgreSQL Fundamentals](https://stanza.dev/courses/postgresql-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*