---
source_course: "postgresql-fundamentals"
source_lesson: "postgresql-fundamentals-views"
---

# Views

## Introduction
A view is a stored query that acts like a virtual table. Instead of writing the same complex query repeatedly, you define it once as a view and query it by name. Views simplify your SQL, improve security, and provide abstraction over your table structure.

## Key Concepts
- **View**: A named, stored SELECT query that acts as a virtual table. It does not store data — it runs the underlying query each time you access it.
- **Materialized View**: A view that physically stores its results. Faster to read but requires manual refreshing to stay current.
- **Updatable View**: A simple view (based on a single table, no aggregates) that allows INSERT, UPDATE, and DELETE through it.
- **WITH CHECK OPTION**: A constraint on updatable views that prevents inserting or updating rows that would not be visible through the view.

## Real World Context
Views are used everywhere in production databases. Security teams create views that expose only certain columns to different roles. Analysts create views for common report queries. Backend developers create views to simplify complex JOINs that multiple API endpoints need. Materialized views power dashboards that need fast reads on expensive aggregation queries.

## Deep Dive

### Creating Views

Define a view with CREATE VIEW and use it like a table:

```sql
-- Simple view
CREATE VIEW published_books AS
SELECT title, author_id, published_year
FROM books
WHERE published_year IS NOT NULL;

-- Use it like a table
SELECT * FROM published_books WHERE published_year > 2000;
```

The view runs its underlying query each time you SELECT from it.

### Complex Views

Views can contain JOINs, CASE expressions, and calculations:

```sql
-- View with joins and calculations
CREATE VIEW book_details AS
SELECT 
    b.id,
    b.title,
    a.name AS author_name,
    b.published_year,
    b.pages,
    CASE 
        WHEN b.pages < 200 THEN 'Short'
        WHEN b.pages < 400 THEN 'Medium'
        ELSE 'Long'
    END AS length_category
FROM books b
JOIN authors a ON b.author_id = a.id;

-- Query the view with simple syntax
SELECT * FROM book_details WHERE length_category = 'Long';
```

This hides the JOIN complexity behind a simple view name.

### OR REPLACE

Modify an existing view without dropping it:

```sql
-- Modify existing view
CREATE OR REPLACE VIEW published_books AS
SELECT title, author_id, published_year, pages
FROM books
WHERE published_year IS NOT NULL;
```

CREATE OR REPLACE only works if the new view has the same columns (or adds new ones). It cannot remove columns.

### Benefits of Views

1. **Simplify complex queries**: Write once, use many times
2. **Security**: Expose only certain columns/rows to users
3. **Abstraction**: Hide table structure changes from applications
4. **Consistency**: Ensure all queries use the same logic

### Updatable Views

Simple views based on a single table can be updated:

```sql
-- This view is updatable
CREATE VIEW active_users AS
SELECT * FROM users WHERE is_active = true;

-- Update through the view
UPDATE active_users SET name = 'Alice Smith' WHERE id = 1;
```

Changes to the view affect the underlying table directly.

### WITH CHECK OPTION

Prevent inserting rows that would be invisible through the view:

```sql
-- Prevent inserting inactive users through active_users view
CREATE VIEW active_users AS
SELECT * FROM users WHERE is_active = true
WITH CHECK OPTION;

-- This fails:
INSERT INTO active_users (name, is_active) VALUES ('Bob', false);
-- ERROR: new row violates check option for view
```

WITH CHECK OPTION enforces that all modifications through the view remain visible through the view.

### Materialized Views

Materialized views store query results physically for faster reads:

```sql
-- Create materialized view
CREATE MATERIALIZED VIEW monthly_stats AS
SELECT 
    DATE_TRUNC('month', created_at) AS month,
    COUNT(*) AS order_count,
    SUM(total) AS revenue
FROM orders
GROUP BY DATE_TRUNC('month', created_at);

-- Refresh when data changes
REFRESH MATERIALIZED VIEW monthly_stats;

-- Refresh without blocking reads
REFRESH MATERIALIZED VIEW CONCURRENTLY monthly_stats;
```

Materialized views trade freshness for speed — the data is only as current as the last refresh.

## Common Pitfalls
1. **Assuming views store data** — Regular views are virtual. They run the underlying query every time, so they are not faster than writing the query directly. Only materialized views store data.
2. **Forgetting to refresh materialized views** — Materialized views do not update automatically. You must call REFRESH MATERIALIZED VIEW manually or via a scheduled job.

## Best Practices
1. **Use views for security and abstraction** — Create views that expose only the columns and rows each role needs.
2. **Use materialized views for expensive aggregations** — If a dashboard query takes seconds to run, materialize it and refresh periodically.
3. **Name views descriptively** — `active_premium_users` is better than `v_users_2`.

## Summary
- Views are stored queries that act like virtual tables — they do not store data.
- Materialized views physically store results for faster reads but require manual refresh.
- Simple views are updatable; WITH CHECK OPTION prevents invisible row modifications.
- CREATE OR REPLACE modifies a view without dropping it.
- Use views for security, abstraction, and simplifying complex queries.

## Code Examples

**A materialized view for dashboard metrics with an index and concurrent refresh to avoid blocking reads**

```sql
-- Materialized view for a dashboard with periodic refresh
CREATE MATERIALIZED VIEW dashboard_stats AS
SELECT 
    DATE_TRUNC('day', created_at) AS day,
    COUNT(*) AS orders,
    SUM(total) AS revenue,
    AVG(total) AS avg_order
FROM orders
GROUP BY DATE_TRUNC('day', created_at);

-- Create index for fast lookups
CREATE INDEX idx_dashboard_stats_day ON dashboard_stats(day);

-- Refresh without blocking reads
REFRESH MATERIALIZED VIEW CONCURRENTLY dashboard_stats;
```


## Resources

- [CREATE VIEW](https://www.postgresql.org/docs/18/sql-createview.html) — Complete VIEW syntax reference

---

> 📘 *This lesson is part of the [PostgreSQL Fundamentals](https://stanza.dev/courses/postgresql-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*