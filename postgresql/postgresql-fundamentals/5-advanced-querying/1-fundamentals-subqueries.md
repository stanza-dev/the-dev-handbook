---
source_course: "postgresql-fundamentals"
source_lesson: "postgresql-fundamentals-subqueries"
---

# Subqueries

A subquery is a query nested inside another query. They help solve complex problems step by step.

## Scalar Subqueries

Return a single value:

```sql
-- Books longer than average
SELECT title, pages
FROM books
WHERE pages > (SELECT AVG(pages) FROM books);

-- Latest order date
SELECT *
FROM orders
WHERE created_at = (SELECT MAX(created_at) FROM orders);
```

## Subqueries with IN

Return a list of values:

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

## Correlated Subqueries

Reference the outer query (executed once per row):

```sql
-- Authors with more than 3 books
SELECT name
FROM authors a
WHERE (
    SELECT COUNT(*) 
    FROM books b 
    WHERE b.author_id = a.id
) > 3;

-- Each book with its author's total book count
SELECT 
    title,
    (SELECT COUNT(*) FROM books b2 WHERE b2.author_id = b.author_id) AS author_books
FROM books b;
```

## EXISTS Subqueries

Check if any rows exist:

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

## Subqueries in FROM (Derived Tables)

```sql
-- Average of averages per category
SELECT AVG(category_avg) AS overall_avg
FROM (
    SELECT category, AVG(price) AS category_avg
    FROM products
    GROUP BY category
) AS category_averages;
```

**Tip**: For complex subqueries, CTEs (next lesson) are often clearer.

📖 [Subqueries](https://www.postgresql.org/docs/18/functions-subquery.html)

---

> 📘 *This lesson is part of the [PostgreSQL Fundamentals](https://stanza.dev/courses/postgresql-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*