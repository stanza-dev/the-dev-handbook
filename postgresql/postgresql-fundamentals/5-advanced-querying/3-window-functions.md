---
source_course: "postgresql-fundamentals"
source_lesson: "postgresql-fundamentals-window-functions"
---

# Window Functions

## Introduction
Window functions perform calculations across a set of rows that are related to the current row, without collapsing them into a single output row like GROUP BY does. They are essential for ranking, running totals, and comparing each row to its neighbors.

## Key Concepts
- **Window function**: A function that operates on a "window" of rows related to the current row, defined by OVER().
- **PARTITION BY**: Divides rows into groups (partitions) for the window function. Like GROUP BY, but without collapsing rows.
- **ORDER BY (in OVER)**: Defines the order of rows within each partition, which determines ranking and running total sequence.
- **ROW_NUMBER / RANK / DENSE_RANK**: Ranking functions that assign a position number to each row.
- **LAG / LEAD**: Access the previous or next row's values within the window.

## Real World Context
Window functions power leaderboards (RANK), "compared to last month" reports (LAG), running totals (SUM OVER), and "top N per category" queries (ROW_NUMBER with PARTITION BY). They are among the most powerful features in SQL and separate intermediate from advanced SQL users.

## Deep Dive

### Basic Syntax

Window functions use the OVER clause to define the window:

```sql
function_name() OVER (
    [PARTITION BY column]
    [ORDER BY column]
)
```

PARTITION BY is optional — without it, the window includes all rows.

### ROW_NUMBER, RANK, DENSE_RANK

These ranking functions assign position numbers to rows:

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

PARTITION BY author_id creates separate numbering sequences for each author.

The three ranking functions differ in how they handle ties:

```sql
-- Given pages: 400, 400, 300, 200
SELECT 
    pages,
    ROW_NUMBER() OVER (ORDER BY pages DESC),  -- 1, 2, 3, 4 (unique)
    RANK() OVER (ORDER BY pages DESC),         -- 1, 1, 3, 4 (gaps after ties)
    DENSE_RANK() OVER (ORDER BY pages DESC)    -- 1, 1, 2, 3 (no gaps)
FROM books;
```

ROW_NUMBER always gives unique numbers. RANK skips numbers after ties. DENSE_RANK never skips.

### Running Totals

SUM with OVER creates running totals:

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

The ORDER BY in OVER makes the SUM accumulate row by row instead of summing everything at once.

### LAG and LEAD

LAG and LEAD access previous and next row values:

```sql
-- Compare to previous month's sales
SELECT 
    month,
    revenue,
    LAG(revenue) OVER (ORDER BY month) AS prev_month,
    revenue - LAG(revenue) OVER (ORDER BY month) AS growth
FROM monthly_sales;
```

LAG looks backward, LEAD looks forward. The first row's LAG returns NULL because there is no previous row.

### FIRST_VALUE

FIRST_VALUE returns the first value in the window:

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

This shows each author's first book alongside all their other books.

### Aggregate Window Functions

Regular aggregates can also be used as window functions:

```sql
-- Each book's percentage of author's total pages
SELECT 
    title,
    pages,
    SUM(pages) OVER (PARTITION BY author_id) AS author_total,
    ROUND(100.0 * pages / SUM(pages) OVER (PARTITION BY author_id), 1) AS pct
FROM books;
```

Unlike GROUP BY, the window function preserves individual rows while computing group-level aggregates.

## Common Pitfalls
1. **Confusing window functions with GROUP BY** — GROUP BY collapses rows into groups. Window functions keep every row and add computed columns alongside them.
2. **Forgetting ORDER BY in OVER for ranking** — ROW_NUMBER, RANK, and DENSE_RANK require ORDER BY in the OVER clause to produce meaningful results.

## Best Practices
1. **Use ROW_NUMBER + PARTITION BY for "top N per group"** — Filter with a subquery: `WHERE rank <= 3` to get the top 3 per group.
2. **Use LAG for period-over-period comparisons** — LAG is the simplest way to compare each row to the previous period.
3. **Name your windows with WINDOW clause** — If you reuse the same OVER definition, define it once with `WINDOW w AS (PARTITION BY ...)` and reference it as `OVER w`.

## Summary
- Window functions compute across related rows without collapsing them.
- PARTITION BY creates groups; ORDER BY defines row sequence within each group.
- ROW_NUMBER gives unique numbers; RANK has gaps after ties; DENSE_RANK has no gaps.
- LAG/LEAD access previous/next row values for comparisons.
- Aggregate functions (SUM, AVG, COUNT) work as window functions when used with OVER.

## Code Examples

**Top-N per group pattern using ROW_NUMBER with PARTITION BY — a very common real-world query**

```sql
-- Top 3 products per category by revenue
SELECT * FROM (
    SELECT 
        category,
        name,
        revenue,
        ROW_NUMBER() OVER (
            PARTITION BY category 
            ORDER BY revenue DESC
        ) AS rank
    FROM products
) ranked
WHERE rank <= 3;
```


## Resources

- [Window Functions](https://www.postgresql.org/docs/18/tutorial-window.html) — Tutorial on window functions

---

> 📘 *This lesson is part of the [PostgreSQL Fundamentals](https://stanza.dev/courses/postgresql-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*