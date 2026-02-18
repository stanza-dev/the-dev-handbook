---
source_course: "postgresql-fundamentals"
source_lesson: "postgresql-fundamentals-views"
---

# Views

A view is a stored query that acts like a virtual table.

## Creating Views

```sql
-- Simple view
CREATE VIEW published_books AS
SELECT title, author_id, published_year
FROM books
WHERE published_year IS NOT NULL;

-- Use it like a table
SELECT * FROM published_books WHERE published_year > 2000;
```

## Complex Views

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

-- Query the view
SELECT * FROM book_details WHERE length_category = 'Long';
```

## OR REPLACE

```sql
-- Modify existing view
CREATE OR REPLACE VIEW published_books AS
SELECT title, author_id, published_year, pages  -- Added pages
FROM books
WHERE published_year IS NOT NULL;
```

## Benefits of Views

1. **Simplify complex queries**: Write once, use many times
2. **Security**: Expose only certain columns/rows
3. **Abstraction**: Hide table structure changes
4. **Consistency**: Ensure all queries use same logic

## Updatable Views

Simple views can be updated:

```sql
-- This view is updatable
CREATE VIEW active_users AS
SELECT * FROM users WHERE is_active = true;

-- Update through the view
UPDATE active_users SET name = 'Alice Smith' WHERE id = 1;
```

## WITH CHECK OPTION

```sql
-- Prevent inserting rows that wouldn't be visible
CREATE VIEW active_users AS
SELECT * FROM users WHERE is_active = true
WITH CHECK OPTION;

-- This fails:
INSERT INTO active_users (name, is_active) VALUES ('Bob', false);
-- ERROR: new row violates check option for view
```

## Materialized Views

Store the query results physically (faster reads, stale data):

```sql
-- Create materialized view
CREATE MATERIALIZED VIEW monthly_stats AS
SELECT 
    DATE_TRUNC('month', created_at) AS month,
    COUNT(*) AS order_count,
    SUM(total) AS revenue
FROM orders
GROUP BY DATE_TRUNC('month', created_at);

-- Refresh when needed
REFRESH MATERIALIZED VIEW monthly_stats;

-- Refresh without blocking reads
REFRESH MATERIALIZED VIEW CONCURRENTLY monthly_stats;
```

📖 [CREATE VIEW](https://www.postgresql.org/docs/18/sql-createview.html)

## Resources

- [CREATE VIEW](https://www.postgresql.org/docs/18/sql-createview.html) — Complete VIEW syntax reference

---

> 📘 *This lesson is part of the [PostgreSQL Fundamentals](https://stanza.dev/courses/postgresql-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*