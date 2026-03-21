---
source_course: "postgresql-fundamentals"
source_lesson: "postgresql-fundamentals-aggregate-functions"
---

# Aggregate Functions

## Introduction
Aggregate functions compute a single result from a set of rows. They are essential for reporting, analytics, and answering questions like "how many", "what's the total", or "what's the average". This lesson covers the core aggregate functions and some PostgreSQL-specific ones.

## Key Concepts
- **COUNT(*)**: Counts all rows, including those with NULL values.
- **COUNT(column)**: Counts only non-NULL values in the specified column.
- **SUM / AVG / MIN / MAX**: Compute the sum, average, minimum, and maximum of numeric columns.
- **STRING_AGG**: Concatenates string values from multiple rows into a single string, with a specified delimiter.
- **ARRAY_AGG**: Collects values from multiple rows into a PostgreSQL array.

## Real World Context
Aggregates power every dashboard and report you have ever seen. "Total revenue this month" uses SUM. "Average order value" uses AVG. "Number of active users" uses COUNT. Without aggregates, you would need to fetch all rows to your application and compute these values in code — much slower and more error-prone.

## Deep Dive

### Common Aggregate Functions

Here is a reference table of the most-used aggregate functions:

| Function | Description |
|----------|-------------|
| `COUNT(*)` | Count all rows |
| `COUNT(column)` | Count non-NULL values |
| `SUM(column)` | Sum of values |
| `AVG(column)` | Average of values |
| `MIN(column)` | Minimum value |
| `MAX(column)` | Maximum value |

### Basic Examples

You can use multiple aggregates in a single query:

```sql
-- Count all books
SELECT COUNT(*) FROM books;

-- Count books with page numbers (non-NULL)
SELECT COUNT(pages) FROM books;

-- Sum, average, min, max in one query
SELECT 
    SUM(pages) AS total_pages,
    AVG(pages) AS avg_pages,
    MIN(pages) AS shortest_book,
    MAX(pages) AS longest_book
FROM books;
```

The difference between `COUNT(*)` and `COUNT(pages)` is important: `COUNT(*)` counts all rows, while `COUNT(pages)` only counts rows where `pages` is not NULL.

### COUNT DISTINCT

Count unique values by combining COUNT with DISTINCT:

```sql
-- Count unique authors
SELECT COUNT(DISTINCT author) FROM books;
```

This returns the number of different authors, not the total number of books.

### Filtering Before Aggregation

Use WHERE to filter rows before aggregating:

```sql
-- Average pages for books after 2000
SELECT AVG(pages) 
FROM books 
WHERE published_year > 2000;
```

WHERE runs before the aggregation, so only matching rows are included in the calculation.

### Finding Rows Matching Aggregates

Use subqueries to compare individual rows against aggregate values:

```sql
-- Find the longest book
SELECT * FROM books 
WHERE pages = (SELECT MAX(pages) FROM books);

-- Find books longer than average
SELECT * FROM books 
WHERE pages > (SELECT AVG(pages) FROM books);
```

The subquery computes the aggregate, and the outer query uses it as a filter.

### String and Array Aggregation

PostgreSQL offers powerful aggregation for non-numeric types:

```sql
-- Concatenate authors into a comma-separated string
SELECT STRING_AGG(DISTINCT author, ', ')
FROM books;
-- Result: 'David Thomas, George Orwell, Martin Fowler, ...'

-- Collect values into an array
SELECT ARRAY_AGG(title) 
FROM books 
WHERE author = 'Robert C. Martin';
-- Result: {'Clean Code', 'Clean Architecture', ...}
```

STRING_AGG and ARRAY_AGG are very useful for building denormalized API responses.

## Common Pitfalls
1. **Using aggregates in WHERE** — You cannot use `WHERE COUNT(*) > 5`. Aggregates work on groups, so use HAVING instead (covered in the GROUP BY lesson).
2. **Confusing COUNT(*) with COUNT(column)** — `COUNT(*)` counts all rows; `COUNT(column)` only counts non-NULL values. This distinction matters when your data has NULLs.

## Best Practices
1. **Use COUNT(*) for total rows, COUNT(column) for non-NULL counts** — Be intentional about which you use based on what you are measuring.
2. **Combine aggregates in one query** — Instead of running separate queries for SUM, AVG, and COUNT, combine them in a single SELECT for better performance.

## Summary
- Aggregate functions (COUNT, SUM, AVG, MIN, MAX) compute single values from sets of rows.
- COUNT(*) counts all rows; COUNT(column) counts only non-NULL values.
- STRING_AGG and ARRAY_AGG aggregate non-numeric values into strings and arrays.
- Use subqueries to compare individual rows against aggregate results.
- Aggregates cannot be used in WHERE — use HAVING instead.

## Code Examples

**A single query that computes all key metrics for a monthly dashboard using multiple aggregate functions**

```sql
-- Dashboard summary query using multiple aggregates
SELECT 
    COUNT(*) AS total_orders,
    COUNT(DISTINCT customer_id) AS unique_customers,
    SUM(total) AS revenue,
    AVG(total) AS avg_order_value,
    MIN(total) AS smallest_order,
    MAX(total) AS largest_order
FROM orders
WHERE created_at >= DATE_TRUNC('month', CURRENT_DATE);
```


## Resources

- [Aggregate Functions](https://www.postgresql.org/docs/18/functions-aggregate.html) — Complete list of aggregate functions

---

> 📘 *This lesson is part of the [PostgreSQL Fundamentals](https://stanza.dev/courses/postgresql-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*