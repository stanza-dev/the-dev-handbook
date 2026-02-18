---
source_course: "postgresql-fundamentals"
source_lesson: "postgresql-fundamentals-update-delete"
---

# Updating and Deleting Data

Learn to modify existing data safely with UPDATE and DELETE.

## UPDATE Syntax

```sql
-- Update specific rows
UPDATE books 
SET pages = 464 
WHERE title = 'Clean Code';

-- Update multiple columns
UPDATE books 
SET 
    published_year = 2020,
    pages = 416
WHERE title = 'The Pragmatic Programmer';

-- Update with expressions
UPDATE products 
SET price = price * 1.10  -- 10% increase
WHERE category = 'electronics';
```

## UPDATE with RETURNING

```sql
-- See what was updated
UPDATE books 
SET pages = 500 
WHERE author = 'Robert C. Martin'
RETURNING id, title, pages;
```

## UPDATE with Subqueries

```sql
-- Update based on another table
UPDATE orders 
SET status = 'shipped'
WHERE customer_id IN (
    SELECT id FROM customers 
    WHERE membership = 'premium'
);
```

## DELETE Syntax

```sql
-- Delete specific rows
DELETE FROM books 
WHERE published_year < 1950;

-- Delete with RETURNING
DELETE FROM sessions 
WHERE created_at < NOW() - INTERVAL '7 days'
RETURNING id, user_id;
```

## ⚠️ Critical Safety Rules

**ALWAYS use WHERE with UPDATE and DELETE!**

```sql
-- DANGEROUS: Updates ALL rows!
UPDATE books SET pages = 100;  -- Every book now has 100 pages

-- DANGEROUS: Deletes ALL rows!
DELETE FROM books;  -- All data gone!

-- SAFE: Always filter
UPDATE books SET pages = 100 WHERE id = 5;
DELETE FROM books WHERE id = 5;
```

## Testing Before Modifying

```sql
-- Step 1: Check what will be affected
SELECT * FROM books WHERE published_year < 1950;

-- Step 2: If it looks right, then delete
DELETE FROM books WHERE published_year < 1950;
```

## Using Transactions for Safety

```sql
-- Start a transaction
BEGIN;

-- Make changes
DELETE FROM books WHERE author = 'Unknown';

-- Check the result
SELECT * FROM books;

-- If everything looks good:
COMMIT;

-- Or if something went wrong:
ROLLBACK;  -- Undo all changes
```

## TRUNCATE for Deleting All Rows

```sql
-- Faster than DELETE for removing all rows
TRUNCATE TABLE logs;

-- Also reset sequences (auto-increment)
TRUNCATE TABLE logs RESTART IDENTITY;
```

📖 [UPDATE Documentation](https://www.postgresql.org/docs/18/sql-update.html)

## Resources

- [DELETE Statement](https://www.postgresql.org/docs/18/sql-delete.html) — Complete DELETE syntax reference

---

> 📘 *This lesson is part of the [PostgreSQL Fundamentals](https://stanza.dev/courses/postgresql-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*