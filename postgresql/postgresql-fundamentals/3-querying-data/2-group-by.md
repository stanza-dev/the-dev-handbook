---
source_course: "postgresql-fundamentals"
source_lesson: "postgresql-fundamentals-group-by"
---

# Grouping Data with GROUP BY

GROUP BY organizes rows into groups based on column values, allowing aggregate calculations per group.

## Basic GROUP BY

```sql
-- Count books per author
SELECT author, COUNT(*) AS book_count
FROM books
GROUP BY author;
```

Output:
```
      author       | book_count
-------------------+------------
 Robert C. Martin  |          3
 Martin Fowler     |          2
 George Orwell     |          1
```

## Important Rule

**Every non-aggregated column in SELECT must be in GROUP BY:**

```sql
-- ✅ Correct: author is in GROUP BY
SELECT author, COUNT(*) FROM books GROUP BY author;

-- ❌ Error: title is not in GROUP BY
SELECT author, title, COUNT(*) FROM books GROUP BY author;
-- ERROR: column "title" must appear in GROUP BY clause
```

## Multiple Grouping Columns

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

## Filtering Groups with HAVING

Use HAVING to filter groups (like WHERE for aggregated data):

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

## WHERE vs HAVING

- **WHERE**: Filters rows BEFORE grouping
- **HAVING**: Filters groups AFTER aggregation

```sql
-- Books after 2000, grouped by author, only authors with 2+ books
SELECT author, COUNT(*) AS book_count
FROM books
WHERE published_year > 2000       -- Filter rows first
GROUP BY author
HAVING COUNT(*) >= 2;             -- Then filter groups
```

## Query Execution Order

```
1. FROM        - Get data from tables
2. WHERE       - Filter rows
3. GROUP BY    - Form groups
4. HAVING      - Filter groups
5. SELECT      - Choose columns
6. ORDER BY    - Sort results
7. LIMIT       - Limit output
```

## Grouping with ROLLUP

```sql
-- Get subtotals and grand total
SELECT 
    author,
    published_year,
    COUNT(*) AS book_count
FROM books
GROUP BY ROLLUP(author, published_year);
```

This adds subtotal rows for each author and a grand total.

📖 [GROUP BY Documentation](https://www.postgresql.org/docs/18/queries-table-expressions.html#QUERIES-GROUP)

---

> 📘 *This lesson is part of the [PostgreSQL Fundamentals](https://stanza.dev/courses/postgresql-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*