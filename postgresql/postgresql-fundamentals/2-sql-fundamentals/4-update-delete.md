---
source_course: "postgresql-fundamentals"
source_lesson: "postgresql-fundamentals-update-delete"
---

# Updating and Deleting Data

## Introduction
UPDATE modifies existing rows and DELETE removes them. These are powerful operations that can affect many rows at once, so using them safely is critical. This lesson covers syntax, safety practices, transactions, and PostgreSQL 18's new RETURNING enhancements.

## Key Concepts
- **UPDATE ... SET**: Modifies column values in existing rows that match a WHERE condition.
- **DELETE FROM**: Removes rows that match a WHERE condition.
- **RETURNING**: Returns the affected rows after an UPDATE or DELETE, so you can see exactly what changed.
- **Transaction (BEGIN/COMMIT/ROLLBACK)**: A group of operations that either all succeed or all fail. Use transactions to preview changes before making them permanent.
- **TRUNCATE**: A fast way to remove all rows from a table without logging individual row deletions.

## Real World Context
Every application needs to update and delete data — changing user settings, removing expired sessions, updating prices. A missing WHERE clause on an UPDATE or DELETE is one of the most common and devastating mistakes a developer can make. Learning transactions and the "SELECT first" pattern prevents data loss.

## Deep Dive

### UPDATE Syntax

UPDATE modifies rows that match the WHERE clause:

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

The WHERE clause determines which rows are affected. Without it, every row in the table is updated.

### UPDATE with RETURNING

RETURNING shows you exactly what was changed:

```sql
-- See what was updated
UPDATE books 
SET pages = 500 
WHERE author = 'Robert C. Martin'
RETURNING id, title, pages;
```

This returns the updated rows immediately, eliminating the need for a separate SELECT.

### PostgreSQL 18: OLD/NEW in RETURNING

PostgreSQL 18 adds the ability to access both old and new values in the RETURNING clause:

```sql
-- PostgreSQL 18: Access old and new values
UPDATE products 
SET price = price * 1.10
WHERE category = 'books'
RETURNING old.price AS old_price, new.price AS new_price;
```

This is invaluable for audit logging — you can see exactly what changed in a single statement without needing triggers or separate queries.

### UPDATE with Subqueries

You can use subqueries to determine which rows to update:

```sql
-- Update based on another table
UPDATE orders 
SET status = 'shipped'
WHERE customer_id IN (
    SELECT id FROM customers 
    WHERE membership = 'premium'
);
```

This updates all orders belonging to premium customers in one operation.

### DELETE Syntax

DELETE removes rows that match the WHERE clause:

```sql
-- Delete specific rows
DELETE FROM books 
WHERE published_year < 1950;

-- Delete with RETURNING
DELETE FROM sessions 
WHERE created_at < NOW() - INTERVAL '7 days'
RETURNING id, user_id;
```

RETURNING works with DELETE too, showing you exactly which rows were removed.

### Critical Safety Rules

Always use WHERE with UPDATE and DELETE. Without it, every row is affected:

```sql
-- DANGEROUS: Updates ALL rows!
UPDATE books SET pages = 100;  -- Every book now has 100 pages

-- DANGEROUS: Deletes ALL rows!
DELETE FROM books;  -- All data gone!

-- SAFE: Always filter
UPDATE books SET pages = 100 WHERE id = 5;
DELETE FROM books WHERE id = 5;
```

The consequences of forgetting WHERE are severe and usually irreversible without backups.

### Testing Before Modifying

Always preview what will be affected before making changes:

```sql
-- Step 1: Check what will be affected
SELECT * FROM books WHERE published_year < 1950;

-- Step 2: If it looks right, then delete
DELETE FROM books WHERE published_year < 1950;
```

This two-step pattern prevents accidental data loss by letting you verify the WHERE clause first.

### Using Transactions for Safety

Transactions let you preview and roll back changes:

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

BEGIN starts a transaction, COMMIT makes changes permanent, and ROLLBACK undoes everything since BEGIN.

### TRUNCATE for Deleting All Rows

When you need to remove all rows, TRUNCATE is much faster than DELETE:

```sql
-- Faster than DELETE for removing all rows
TRUNCATE TABLE logs;

-- Also reset sequences (auto-increment)
TRUNCATE TABLE logs RESTART IDENTITY;
```

TRUNCATE does not scan individual rows — it simply deallocates the table's data pages, making it nearly instant even on large tables.

## Common Pitfalls
1. **Forgetting the WHERE clause** — An UPDATE or DELETE without WHERE affects every row in the table. Always write the WHERE clause first, then add the SET or DELETE.
2. **Not testing with SELECT first** — Run your WHERE clause in a SELECT before using it in UPDATE or DELETE to verify you are targeting the correct rows.
3. **Not using transactions for risky changes** — Wrap UPDATE and DELETE operations in BEGIN/ROLLBACK transactions when you are unsure, so you can undo if something goes wrong.

## Best Practices
1. **Always preview with SELECT** — Before running UPDATE or DELETE, run the same WHERE clause in a SELECT to verify which rows will be affected.
2. **Use RETURNING to confirm changes** — RETURNING shows you exactly what was modified, providing an immediate audit trail.
3. **Wrap risky operations in transactions** — Use BEGIN/COMMIT/ROLLBACK to create a safety net for important data modifications.

## Summary
- UPDATE modifies existing rows; DELETE removes them. Both require a WHERE clause.
- RETURNING shows the affected rows immediately after UPDATE or DELETE.
- PostgreSQL 18 adds OLD/NEW references in RETURNING to see before and after values.
- Always preview changes with SELECT before running UPDATE or DELETE.
- Use transactions (BEGIN/COMMIT/ROLLBACK) as a safety net for risky modifications.
- TRUNCATE is faster than DELETE for removing all rows from a table.

## Code Examples

**Safe update using a transaction with PG18's OLD/NEW RETURNING to see both old and new prices before committing**

```sql
-- Safe update pattern with transaction and RETURNING
BEGIN;

UPDATE products
SET price = price * 1.10
WHERE category = 'books'
RETURNING id, name, old.price AS old_price, new.price AS new_price;
-- Review the output...

-- If correct:
COMMIT;
-- If wrong:
-- ROLLBACK;
```

**DELETE with RETURNING — removes expired sessions and returns the deleted rows for logging**

```sql
-- Delete expired sessions and log what was removed
DELETE FROM sessions
WHERE created_at < NOW() - INTERVAL '30 days'
RETURNING id, user_id, created_at;
```


## Resources

- [DELETE Statement](https://www.postgresql.org/docs/18/sql-delete.html) — Complete DELETE syntax reference
- [UPDATE Statement](https://www.postgresql.org/docs/18/sql-update.html) — Complete UPDATE syntax reference

---

> 📘 *This lesson is part of the [PostgreSQL Fundamentals](https://stanza.dev/courses/postgresql-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*