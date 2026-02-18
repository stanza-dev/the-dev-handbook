---
source_course: "postgresql-fundamentals"
source_lesson: "postgresql-fundamentals-aggregate-functions"
---

# Aggregate Functions

Aggregate functions compute a single result from a set of rows. They're essential for reporting and analytics.

## Common Aggregate Functions

| Function | Description |
|----------|-------------|
| `COUNT(*)` | Count all rows |
| `COUNT(column)` | Count non-NULL values |
| `SUM(column)` | Sum of values |
| `AVG(column)` | Average of values |
| `MIN(column)` | Minimum value |
| `MAX(column)` | Maximum value |

## Basic Examples

```sql
-- Count all books
SELECT COUNT(*) FROM books;

-- Count books with page numbers
SELECT COUNT(pages) FROM books;

-- Sum, average, min, max
SELECT 
    SUM(pages) AS total_pages,
    AVG(pages) AS avg_pages,
    MIN(pages) AS shortest_book,
    MAX(pages) AS longest_book
FROM books;
```

## COUNT DISTINCT

```sql
-- Count unique authors
SELECT COUNT(DISTINCT author) FROM books;
```

## Filtering with Aggregates

You can't use aggregates in WHERE, but you can filter first:

```sql
-- Average pages for books after 2000
SELECT AVG(pages) 
FROM books 
WHERE published_year > 2000;
```

## Finding Rows Matching Aggregates

Use subqueries to find rows matching aggregate values:

```sql
-- Find the longest book
SELECT * FROM books 
WHERE pages = (SELECT MAX(pages) FROM books);

-- Find books longer than average
SELECT * FROM books 
WHERE pages > (SELECT AVG(pages) FROM books);
```

## String Aggregation

```sql
-- Concatenate authors into a single string
SELECT STRING_AGG(DISTINCT author, ', ')
FROM books;
-- Result: 'David Thomas, George Orwell, Martin Fowler, ...'

-- With ordering
SELECT STRING_AGG(author, ', ' ORDER BY author)
FROM books;
```

## Array Aggregation

```sql
-- Collect values into an array
SELECT ARRAY_AGG(title) 
FROM books 
WHERE author = 'Robert C. Martin';
-- Result: {'Clean Code', 'Clean Architecture', ...}
```

📖 [Aggregate Functions Documentation](https://www.postgresql.org/docs/18/functions-aggregate.html)

## Resources

- [Aggregate Functions](https://www.postgresql.org/docs/18/functions-aggregate.html) — Complete list of aggregate functions

---

> 📘 *This lesson is part of the [PostgreSQL Fundamentals](https://stanza.dev/courses/postgresql-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*