---
source_course: "postgresql-fundamentals"
source_lesson: "postgresql-fundamentals-distinct-expressions"
---

# DISTINCT and Calculated Columns

## Introduction
Beyond basic SELECT, PostgreSQL provides powerful features for transforming data: removing duplicates with DISTINCT, conditional logic with CASE, and a rich library of built-in functions. These tools let you shape query results without changing the underlying data.

## Key Concepts
- **DISTINCT**: Removes duplicate rows from the result set. Only unique combinations of the selected columns are returned.
- **DISTINCT ON**: A PostgreSQL-specific feature that returns the first row for each distinct value of the specified columns, based on the ORDER BY clause.
- **CASE expression**: SQL's if-then-else logic. It returns different values based on conditions, useful for categorizing data in queries.
- **Built-in functions**: PostgreSQL provides hundreds of functions for string manipulation, date arithmetic, math operations, and more.

## Real World Context
DISTINCT ON is a PostgreSQL superpower — it solves "get the latest record per group" problems in a single query, which would require window functions or subqueries in other databases. CASE expressions are used everywhere from building report columns to conditional aggregation.

## Deep Dive

### Removing Duplicates with DISTINCT

DISTINCT eliminates duplicate rows from results:

```sql
-- Unique authors
SELECT DISTINCT author FROM books;

-- Unique combinations
SELECT DISTINCT author, published_year FROM books;
```

With multiple columns, DISTINCT applies to the combination — each unique (author, published_year) pair appears once.

### DISTINCT ON (PostgreSQL-specific)

DISTINCT ON returns the first row for each group based on ORDER BY:

```sql
-- Most recent book by each author
SELECT DISTINCT ON (author) 
    author, title, published_year
FROM books
ORDER BY author, published_year DESC;
```

This returns one row per author — the one with the highest published_year. It is much simpler than the equivalent window function approach.

### Calculated Columns

You can perform arithmetic and string operations directly in SELECT:

```sql
-- Arithmetic expressions
SELECT 
    title,
    price,
    price * 0.9 AS discounted_price,
    price * 1.2 AS with_tax
FROM products;

-- String concatenation
SELECT 
    first_name || ' ' || last_name AS full_name
FROM users;
```

The `||` operator concatenates strings in PostgreSQL.

### CASE Expressions

CASE provides conditional logic in queries:

```sql
-- Categorize books by length
SELECT 
    title,
    pages,
    CASE 
        WHEN pages < 200 THEN 'Short'
        WHEN pages < 400 THEN 'Medium'
        ELSE 'Long'
    END AS length_category
FROM books;

-- Conditional aggregation
SELECT 
    COUNT(*) AS total_books,
    COUNT(CASE WHEN published_year >= 2000 THEN 1 END) AS modern_books,
    COUNT(CASE WHEN pages > 500 THEN 1 END) AS long_books
FROM books;
```

CASE inside COUNT is a powerful pattern for computing multiple conditional counts in a single query.

### String Functions

PostgreSQL provides many string manipulation functions:

```sql
SELECT 
    UPPER(title) AS uppercase_title,
    LOWER(author) AS lowercase_author,
    LENGTH(title) AS title_length,
    SUBSTRING(title, 1, 20) AS short_title,
    TRIM('  text  ') AS trimmed,
    REPLACE(title, 'Code', 'Software') AS replaced
FROM books;
```

These functions return transformed values without modifying the stored data.

### Date Functions

Date functions are essential for time-based queries:

```sql
SELECT 
    created_at,
    DATE(created_at) AS date_only,
    EXTRACT(YEAR FROM created_at) AS year,
    EXTRACT(MONTH FROM created_at) AS month,
    AGE(created_at) AS time_since,
    created_at + INTERVAL '30 days' AS plus_30_days
FROM users;
```

EXTRACT pulls individual components from timestamps. AGE calculates the difference between a timestamp and now.

### Math Functions

Common math functions for numerical operations:

```sql
SELECT 
    ROUND(price, 2) AS rounded,
    CEIL(price) AS ceiling,
    FLOOR(price) AS floor,
    ABS(-10) AS absolute_value,
    POWER(2, 10) AS two_to_ten
FROM products;
```

ROUND is especially useful for formatting monetary values in reports.

## Common Pitfalls
1. **Using DISTINCT ON without ORDER BY** — DISTINCT ON requires ORDER BY to determine which row is "first" for each group. Without it, the result is unpredictable.
2. **Forgetting ELSE in CASE** — If no WHEN condition matches and there is no ELSE, CASE returns NULL. Always include ELSE unless you intentionally want NULL.

## Best Practices
1. **Use DISTINCT ON for "latest per group" queries** — It is cleaner and often faster than window functions or correlated subqueries for this common pattern.
2. **Use CASE for conditional aggregation** — Instead of running separate queries for different categories, use CASE inside aggregate functions to compute everything in one pass.

## Summary
- DISTINCT removes duplicate rows; DISTINCT ON returns the first row per group.
- CASE provides if-then-else logic directly in queries.
- PostgreSQL has rich built-in functions for strings (UPPER, LENGTH, REPLACE), dates (EXTRACT, AGE), and math (ROUND, CEIL, FLOOR).
- Use `||` for string concatenation and arithmetic operators for calculated columns.
- DISTINCT ON is a PostgreSQL-specific feature that simplifies "first per group" queries.

## Code Examples

**DISTINCT ON to get the latest order for each customer — a common pattern that replaces complex window function queries**

```sql
-- DISTINCT ON: get the most recent order per customer
SELECT DISTINCT ON (customer_id)
    customer_id,
    id AS order_id,
    total,
    created_at
FROM orders
ORDER BY customer_id, created_at DESC;
```


## Resources

- [Functions and Operators](https://www.postgresql.org/docs/18/functions.html) — Complete reference for PostgreSQL built-in functions and operators

---

> 📘 *This lesson is part of the [PostgreSQL Fundamentals](https://stanza.dev/courses/postgresql-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*