---
source_course: "postgresql-fundamentals"
source_lesson: "postgresql-fundamentals-window-functions"
---

# Window Functions

Window functions perform calculations across related rows without collapsing them into groups.

## Basic Syntax

```sql
function_name() OVER (
    [PARTITION BY column]
    [ORDER BY column]
)
```

## ROW_NUMBER, RANK, DENSE_RANK

```sql
-- Number each book by publication year
SELECT 
    title,
    published_year,
    ROW_NUMBER() OVER (ORDER BY published_year) AS row_num
FROM books;

-- Rank within each author's books
SELECT 
    author_id,
    title,
    pages,
    ROW_NUMBER() OVER (PARTITION BY author_id ORDER BY pages DESC) AS rank
FROM books;
```

### Difference between ranking functions:

```sql
-- Given pages: 400, 400, 300, 200
SELECT 
    pages,
    ROW_NUMBER() OVER (ORDER BY pages DESC),  -- 1, 2, 3, 4 (unique)
    RANK() OVER (ORDER BY pages DESC),         -- 1, 1, 3, 4 (gaps)
    DENSE_RANK() OVER (ORDER BY pages DESC)    -- 1, 1, 2, 3 (no gaps)
FROM books;
```

## Running Totals

```sql
-- Cumulative sales by date
SELECT 
    date,
    amount,
    SUM(amount) OVER (ORDER BY date) AS running_total
FROM sales;

-- Running total per category
SELECT 
    category,
    date,
    amount,
    SUM(amount) OVER (
        PARTITION BY category 
        ORDER BY date
    ) AS category_running_total
FROM sales;
```

## LAG and LEAD

Access previous/next row values:

```sql
-- Compare to previous month's sales
SELECT 
    month,
    revenue,
    LAG(revenue) OVER (ORDER BY month) AS prev_month,
    revenue - LAG(revenue) OVER (ORDER BY month) AS growth
FROM monthly_sales;

-- Next book by same author
SELECT 
    author_id,
    title,
    published_year,
    LEAD(title) OVER (PARTITION BY author_id ORDER BY published_year) AS next_book
FROM books;
```

## FIRST_VALUE, LAST_VALUE, NTH_VALUE

```sql
-- Compare each book to the author's first book
SELECT 
    title,
    published_year,
    FIRST_VALUE(title) OVER (
        PARTITION BY author_id 
        ORDER BY published_year
    ) AS debut_book
FROM books;
```

## Aggregate Window Functions

```sql
-- Each book's percentage of author's total pages
SELECT 
    title,
    pages,
    SUM(pages) OVER (PARTITION BY author_id) AS author_total,
    ROUND(100.0 * pages / SUM(pages) OVER (PARTITION BY author_id), 1) AS pct
FROM books;
```

📖 [Window Functions](https://www.postgresql.org/docs/18/functions-window.html)

## Resources

- [Window Functions](https://www.postgresql.org/docs/18/tutorial-window.html) — Tutorial on window functions

---

> 📘 *This lesson is part of the [PostgreSQL Fundamentals](https://stanza.dev/courses/postgresql-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*