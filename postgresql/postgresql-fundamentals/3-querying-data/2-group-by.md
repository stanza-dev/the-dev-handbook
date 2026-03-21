---
source_course: "postgresql-fundamentals"
source_lesson: "postgresql-fundamentals-group-by"
---

# Grouping Data with GROUP BY

## Introduction
GROUP BY organizes rows into groups based on column values, allowing aggregate calculations per group. While a plain aggregate gives you one result for the entire table, GROUP BY gives you one result per distinct value — enabling breakdowns like "sales by region" or "books per author".

## Key Concepts
- **GROUP BY**: Divides rows into groups based on one or more columns. Each group produces one row in the output.
- **HAVING**: Filters groups after aggregation, like WHERE for groups. Use HAVING when you need to filter by an aggregate value.
- **Query Execution Order**: FROM -> WHERE -> GROUP BY -> HAVING -> SELECT -> ORDER BY -> LIMIT. Understanding this order explains why you cannot use aggregates in WHERE.
- **ROLLUP**: A GROUP BY modifier that adds subtotal and grand total rows to the output.

## Real World Context
Every report with a breakdown uses GROUP BY: revenue by product category, users by country, errors by service. HAVING lets you filter these breakdowns — for example, showing only categories with more than $10,000 in sales. These are the queries that power business intelligence dashboards.

## Deep Dive

### Basic GROUP BY

GROUP BY creates one output row per distinct value in the grouped column:

```sql
-- Count books per author
SELECT author, COUNT(*) AS book_count
FROM books
GROUP BY author;
```

This produces one row per author, with the count of their books.

### The GROUP BY Rule

Every non-aggregated column in SELECT must appear in GROUP BY:

```sql
-- Correct: author is in GROUP BY
SELECT author, COUNT(*) FROM books GROUP BY author;

-- Error: title is not in GROUP BY
SELECT author, title, COUNT(*) FROM books GROUP BY author;
-- ERROR: column "title" must appear in GROUP BY clause
```

This rule exists because each group produces one row, and PostgreSQL would not know which title to show if an author has multiple books.

### Multiple Grouping Columns

You can group by multiple columns for finer breakdowns:

```sql
-- Count books per author per decade
SELECT 
    author,
    (published_year / 10) * 10 AS decade,
    COUNT(*) AS book_count
FROM books
GROUP BY author, (published_year / 10) * 10
ORDER BY author, decade;
```

Each unique combination of author and decade produces one row.

### Filtering Groups with HAVING

HAVING filters groups after aggregation:

```sql
-- Authors with more than 2 books
SELECT author, COUNT(*) AS book_count
FROM books
GROUP BY author
HAVING COUNT(*) > 2;

-- Categories with average price over $50
SELECT category, AVG(price) AS avg_price
FROM products
GROUP BY category
HAVING AVG(price) > 50;
```

HAVING works on the aggregated results, not individual rows.

### WHERE vs HAVING

The key difference is timing — WHERE filters rows before grouping, HAVING filters groups after aggregation:

```sql
-- Books after 2000, grouped by author, only authors with 2+ books
SELECT author, COUNT(*) AS book_count
FROM books
WHERE published_year > 2000       -- Filter rows first
GROUP BY author
HAVING COUNT(*) >= 2;             -- Then filter groups
```

WHERE reduces the input rows; HAVING reduces the output groups.

### Query Execution Order

Understanding this order explains why aggregates work in HAVING but not WHERE:

```
1. FROM        - Get data from tables
2. WHERE       - Filter rows
3. GROUP BY    - Form groups
4. HAVING      - Filter groups
5. SELECT      - Choose columns
6. ORDER BY    - Sort results
7. LIMIT       - Limit output
```

Aggregates are computed during step 3-4, so they are available in HAVING (step 4) but not in WHERE (step 2).

### Grouping with ROLLUP

ROLLUP adds subtotal and grand total rows:

```sql
-- Get subtotals and grand total
SELECT 
    author,
    published_year,
    COUNT(*) AS book_count
FROM books
GROUP BY ROLLUP(author, published_year);
```

This adds subtotal rows for each author (with published_year as NULL) and a grand total row (with both columns as NULL).

## Common Pitfalls
1. **Selecting non-aggregated columns not in GROUP BY** — This always causes an error. Every column in SELECT must either be in GROUP BY or inside an aggregate function.
2. **Using WHERE instead of HAVING for aggregate conditions** — `WHERE COUNT(*) > 5` fails because WHERE runs before GROUP BY. Use `HAVING COUNT(*) > 5` instead.

## Best Practices
1. **Filter with WHERE before GROUP BY when possible** — Reducing the number of rows before grouping is more efficient than filtering groups with HAVING.
2. **Use meaningful aliases** — `COUNT(*) AS book_count` is much clearer than just `COUNT(*)` in query results.
3. **Remember the execution order** — When debugging GROUP BY queries, think through the execution order: FROM -> WHERE -> GROUP BY -> HAVING -> SELECT -> ORDER BY.

## Summary
- GROUP BY creates one output row per distinct value in the grouped columns.
- Every non-aggregated column in SELECT must be in GROUP BY.
- HAVING filters groups after aggregation; WHERE filters rows before grouping.
- Query execution order: FROM -> WHERE -> GROUP BY -> HAVING -> SELECT -> ORDER BY -> LIMIT.
- ROLLUP adds subtotals and grand totals to GROUP BY results.

## Code Examples

**GROUP BY with WHERE (pre-filter), HAVING (post-filter), and multiple aggregates for a sales report**

```sql
-- Sales report by category with HAVING filter
SELECT 
    category,
    COUNT(*) AS product_count,
    SUM(price * quantity) AS total_revenue,
    AVG(price) AS avg_price
FROM products
WHERE is_active = true            -- Filter rows first
GROUP BY category
HAVING SUM(price * quantity) > 1000  -- Then filter groups
ORDER BY total_revenue DESC;
```


## Resources

- [GROUP BY and HAVING](https://www.postgresql.org/docs/18/queries-table-expressions.html#QUERIES-GROUP) — Official documentation for GROUP BY and HAVING clauses

---

> 📘 *This lesson is part of the [PostgreSQL Fundamentals](https://stanza.dev/courses/postgresql-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*