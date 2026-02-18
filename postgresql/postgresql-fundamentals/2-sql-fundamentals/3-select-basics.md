---
source_course: "postgresql-fundamentals"
source_lesson: "postgresql-fundamentals-select-basics"
---

# Querying Data with SELECT

The `SELECT` statement retrieves data from tables. It's the most frequently used SQL command.

## Basic SELECT Syntax

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

## Filtering with WHERE

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

## Pattern Matching with LIKE

```sql
-- % matches any sequence of characters
SELECT * FROM books WHERE title LIKE 'The%';      -- starts with 'The'
SELECT * FROM books WHERE title LIKE '%SQL%';     -- contains 'SQL'
SELECT * FROM books WHERE author LIKE '%Martin';  -- ends with 'Martin'

-- _ matches exactly one character
SELECT * FROM books WHERE title LIKE '____';      -- exactly 4 characters

-- Case-insensitive matching
SELECT * FROM books WHERE title ILIKE '%sql%';
```

## Range and Set Operators

```sql
-- BETWEEN (inclusive)
SELECT * FROM books 
WHERE published_year BETWEEN 2000 AND 2020;

-- IN (set membership)
SELECT * FROM books 
WHERE published_year IN (2008, 2018, 2019);

-- NOT IN
SELECT * FROM books 
WHERE author NOT IN ('Unknown', 'Anonymous');
```

## NULL Handling

```sql
-- Check for NULL (not = NULL!)
SELECT * FROM books WHERE pages IS NULL;
SELECT * FROM books WHERE pages IS NOT NULL;

-- COALESCE: return first non-NULL value
SELECT 
    title,
    COALESCE(pages, 0) AS pages  -- 0 if pages is NULL
FROM books;
```

## Sorting with ORDER BY

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

## Limiting Results

```sql
-- Get first 10 rows
SELECT * FROM books LIMIT 10;

-- Skip rows (pagination)
SELECT * FROM books LIMIT 10 OFFSET 20;  -- Page 3

-- SQL standard alternative
SELECT * FROM books 
FETCH FIRST 10 ROWS ONLY;
```

📖 [SELECT Documentation](https://www.postgresql.org/docs/18/sql-select.html)

---

> 📘 *This lesson is part of the [PostgreSQL Fundamentals](https://stanza.dev/courses/postgresql-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*