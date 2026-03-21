---
source_course: "postgresql-fundamentals"
source_lesson: "postgresql-fundamentals-select-basics"
---

# Querying Data with SELECT

## Introduction
The `SELECT` statement retrieves data from tables and is the most frequently used SQL command. Whether you are building a dashboard, an API endpoint, or analyzing data, you will use SELECT constantly. This lesson covers filtering, pattern matching, sorting, and pagination.

## Key Concepts
- **WHERE clause**: Filters rows based on conditions. Only rows that satisfy the condition are returned.
- **LIKE / ILIKE**: Pattern matching operators. `%` matches any sequence of characters, `_` matches exactly one character. ILIKE is case-insensitive (PostgreSQL-specific).
- **ORDER BY**: Sorts the result set by one or more columns, ascending (ASC) or descending (DESC).
- **LIMIT / OFFSET**: Controls pagination by restricting how many rows are returned and skipping a number of rows.
- **COALESCE**: Returns the first non-NULL value from a list of arguments — useful for providing default values.

## Real World Context
Every API endpoint that returns data uses SELECT under the hood. Knowing how to filter efficiently, sort correctly, and paginate results is fundamental to building performant applications. Poor WHERE clauses lead to full table scans; missing ORDER BY leads to unpredictable result ordering.

## Deep Dive

### Basic SELECT Syntax

The simplest SELECT retrieves all or specific columns from a table:

```sql
-- Select all columns
SELECT * FROM books;

-- Select specific columns
SELECT title, author FROM books;

-- Rename columns in output (aliases)
SELECT 
    title AS book_title,
    author AS written_by
FROM books;
```

Aliases with `AS` rename columns in the output without changing the table. This is useful for making results more readable.

### Filtering with WHERE

WHERE filters rows before they are returned. You can use comparison operators and combine conditions:

```sql
-- Equality
SELECT * FROM books WHERE author = 'George Orwell';

-- Comparison operators
SELECT * FROM books WHERE published_year > 2000;
SELECT * FROM books WHERE pages >= 300;
SELECT * FROM books WHERE published_year <> 1984;  -- not equal

-- Multiple conditions
SELECT * FROM books 
WHERE published_year > 2000 
  AND pages < 500;

SELECT * FROM books 
WHERE author = 'Robert C. Martin' 
   OR author = 'Martin Fowler';
```

AND requires both conditions to be true; OR requires at least one.

### Pattern Matching with LIKE

LIKE lets you match patterns in text columns:

```sql
-- % matches any sequence of characters
SELECT * FROM books WHERE title LIKE 'The%';      -- starts with 'The'
SELECT * FROM books WHERE title LIKE '%SQL%';     -- contains 'SQL'

-- _ matches exactly one character
SELECT * FROM books WHERE title LIKE '____';      -- exactly 4 characters

-- Case-insensitive matching (PostgreSQL-specific)
SELECT * FROM books WHERE title ILIKE '%sql%';
```

ILIKE is a PostgreSQL extension that makes pattern matching case-insensitive — it is not available in all databases.

### Range and Set Operators

BETWEEN and IN simplify common filtering patterns:

```sql
-- BETWEEN (inclusive)
SELECT * FROM books 
WHERE published_year BETWEEN 2000 AND 2020;

-- IN (set membership)
SELECT * FROM books 
WHERE published_year IN (2008, 2018, 2019);
```

BETWEEN is inclusive on both ends. IN is equivalent to multiple OR conditions.

### NULL Handling

NULL represents an unknown or missing value. You cannot use `=` to check for NULL:

```sql
-- Check for NULL (not = NULL!)
SELECT * FROM books WHERE pages IS NULL;
SELECT * FROM books WHERE pages IS NOT NULL;

-- COALESCE: return first non-NULL value
SELECT 
    title,
    COALESCE(pages, 0) AS pages
FROM books;
```

COALESCE returns the first non-NULL argument. Here it replaces NULL pages with 0.

### Sorting with ORDER BY

ORDER BY controls the sort order of results:

```sql
-- Ascending (default)
SELECT * FROM books ORDER BY published_year;

-- Descending
SELECT * FROM books ORDER BY published_year DESC;

-- Multiple columns
SELECT * FROM books 
ORDER BY author ASC, published_year DESC;

-- NULLs positioning
SELECT * FROM books ORDER BY pages NULLS LAST;
```

NULLS LAST ensures NULL values appear at the end, which is usually what you want.

### Limiting Results

LIMIT and OFFSET control pagination:

```sql
-- Get first 10 rows
SELECT * FROM books LIMIT 10;

-- Skip rows (pagination)
SELECT * FROM books LIMIT 10 OFFSET 20;  -- Page 3

-- SQL standard alternative
SELECT * FROM books 
FETCH FIRST 10 ROWS ONLY;
```

For large offsets, consider keyset pagination instead — OFFSET becomes slow on large tables.

## Common Pitfalls
1. **Using `= NULL` instead of `IS NULL`** — NULL is not a value, it represents the absence of a value. `WHERE column = NULL` always returns zero rows. Use `IS NULL` instead.
2. **Forgetting ORDER BY with LIMIT** — Without ORDER BY, the rows returned by LIMIT are unpredictable. Always sort before limiting.
3. **Using OFFSET for deep pagination** — `OFFSET 1000000` requires scanning and discarding a million rows. Use keyset pagination (`WHERE id > last_id LIMIT 10`) for large datasets.

## Best Practices
1. **Select only the columns you need** — `SELECT *` retrieves all columns, which wastes bandwidth and prevents index-only scans. List specific columns instead.
2. **Always pair LIMIT with ORDER BY** — Without ORDER BY, results are in an undefined order and may change between queries.
3. **Use ILIKE for user-facing search** — Users expect case-insensitive search. ILIKE handles this without forcing you to call LOWER() on both sides.

## Summary
- SELECT retrieves data; use aliases to rename columns in the output.
- WHERE filters rows with comparison operators, AND, OR, LIKE, BETWEEN, and IN.
- Use `IS NULL` (not `= NULL`) to check for missing values.
- ORDER BY sorts results; always use it with LIMIT.
- COALESCE provides default values for NULL columns.

## Code Examples

**Standard pagination query with filtering, sorting, and LIMIT/OFFSET**

```sql
-- Pagination pattern with ORDER BY
SELECT id, title, author, published_year
FROM books
WHERE published_year > 2000
ORDER BY published_year DESC, title ASC
LIMIT 10 OFFSET 0;  -- Page 1
-- For page 2: OFFSET 10
-- For page 3: OFFSET 20
```


## Resources

- [SELECT Statement](https://www.postgresql.org/docs/18/sql-select.html) — Complete SELECT syntax reference

---

> 📘 *This lesson is part of the [PostgreSQL Fundamentals](https://stanza.dev/courses/postgresql-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*