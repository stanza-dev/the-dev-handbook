---
source_course: "postgresql-fundamentals"
source_lesson: "postgresql-fundamentals-distinct-expressions"
---

# DISTINCT and Calculated Columns

## Removing Duplicates with DISTINCT

```sql
-- Unique authors
SELECT DISTINCT author FROM books;

-- Unique combinations
SELECT DISTINCT author, published_year FROM books;
```

## DISTINCT ON (PostgreSQL-specific)

Get the first row for each group:

```sql
-- Most recent book by each author
SELECT DISTINCT ON (author) 
    author, title, published_year
FROM books
ORDER BY author, published_year DESC;
```

## Calculated Columns

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

## CASE Expressions

Conditional logic in queries:

```sql
-- Simple CASE
SELECT 
    title,
    pages,
    CASE 
        WHEN pages < 200 THEN 'Short'
        WHEN pages < 400 THEN 'Medium'
        ELSE 'Long'
    END AS length_category
FROM books;

-- CASE for conditional aggregation
SELECT 
    COUNT(*) AS total_books,
    COUNT(CASE WHEN published_year >= 2000 THEN 1 END) AS modern_books,
    COUNT(CASE WHEN pages > 500 THEN 1 END) AS long_books
FROM books;
```

## Built-in Functions

### String Functions
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

### Date Functions
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

### Math Functions
```sql
SELECT 
    ROUND(price, 2) AS rounded,
    CEIL(price) AS ceiling,
    FLOOR(price) AS floor,
    ABS(-10) AS absolute_value,
    POWER(2, 10) AS two_to_ten
FROM products;
```

📖 [Functions and Operators](https://www.postgresql.org/docs/18/functions.html)

---

> 📘 *This lesson is part of the [PostgreSQL Fundamentals](https://stanza.dev/courses/postgresql-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*