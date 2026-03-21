---
source_course: "postgresql-fundamentals"
source_lesson: "postgresql-fundamentals-insert-data"
---

# Inserting Data with INSERT

## Introduction
The `INSERT` statement adds new rows to a table. It is one of the four fundamental SQL operations (INSERT, SELECT, UPDATE, DELETE). This lesson covers everything from basic single-row inserts to advanced upsert patterns with ON CONFLICT.

## Key Concepts
- **INSERT INTO**: The SQL command to add new rows to a table.
- **RETURNING**: A PostgreSQL clause that returns data from the inserted rows, commonly used to get auto-generated IDs.
- **ON CONFLICT (UPSERT)**: Insert a row or update it if a conflict occurs on a unique constraint. This is PostgreSQL's way of doing "insert or update".
- **EXCLUDED**: A special reference in ON CONFLICT that refers to the values that were proposed for insertion.

## Real World Context
Every web application inserts data constantly — user registrations, form submissions, log entries, API data imports. The RETURNING clause is especially valuable because it eliminates the need for a separate SELECT query to get the auto-generated ID after insertion.

## Deep Dive

### Basic INSERT Syntax

The simplest form lists the columns and values to insert:

```sql
-- Insert a single row
INSERT INTO books (title, author, published_year, pages)
VALUES ('The Pragmatic Programmer', 'David Thomas', 2019, 352);
```

You specify only the columns you want to fill. Any column with a DEFAULT value or that allows NULL will be handled automatically.

### Inserting Multiple Rows

Insert several rows in a single statement for better performance:

```sql
INSERT INTO books (title, author, published_year, pages)
VALUES 
    ('Clean Code', 'Robert C. Martin', 2008, 464),
    ('Design Patterns', 'Gang of Four', 1994, 395),
    ('Refactoring', 'Martin Fowler', 2018, 448);
```

This is significantly faster than three separate INSERT statements because it requires only one round-trip to the database.

### INSERT with RETURNING

The RETURNING clause lets you get back data from the inserted rows:

```sql
INSERT INTO books (title, author, published_year)
VALUES ('PostgreSQL Up and Running', 'Regina Obe', 2022)
RETURNING id, title, created_at;
```

This returns the auto-generated `id` and `created_at` values immediately, without needing a separate SELECT.

### INSERT from SELECT (Copying Data)

You can insert rows from the result of a query:

```sql
-- Copy data from another table
INSERT INTO archived_books (title, author, published_year)
SELECT title, author, published_year
FROM books
WHERE published_year < 2000;
```

This is useful for archiving, data migration, or populating tables from existing data.

### Handling Conflicts (UPSERT)

ON CONFLICT lets you handle duplicate key violations gracefully:

```sql
-- Insert or update if exists (ON CONFLICT)
INSERT INTO products (sku, name, price)
VALUES ('ABC123', 'Widget', 29.99)
ON CONFLICT (sku) 
DO UPDATE SET 
    price = EXCLUDED.price,
    updated_at = NOW();

-- Or do nothing on conflict
INSERT INTO products (sku, name, price)
VALUES ('ABC123', 'Widget', 29.99)
ON CONFLICT (sku) DO NOTHING;
```

The `EXCLUDED` keyword refers to the row that was proposed for insertion. This pattern is commonly used for idempotent API endpoints.

## Common Pitfalls
1. **Forgetting column order matters** — When inserting without column names (`INSERT INTO books VALUES (...)`), values must match the exact column order in the table. Always specify column names explicitly.
2. **Ignoring RETURNING** — Many developers issue a separate SELECT after INSERT to get the generated ID. Use RETURNING instead — it is faster and atomic.

## Best Practices
1. **Always list column names explicitly** — `INSERT INTO books (title, author)` is clearer and safer than `INSERT INTO books VALUES (...)`. It also protects against schema changes.
2. **Use multi-row INSERT for bulk data** — Inserting many rows in a single statement is much faster than individual inserts.
3. **Use ON CONFLICT for idempotent operations** — This prevents duplicate key errors and makes your API endpoints safe to retry.

## Summary
- INSERT adds rows with explicit column names and VALUES.
- Multi-row INSERT is faster than multiple single-row inserts.
- RETURNING eliminates the need for a separate SELECT to get generated values.
- ON CONFLICT provides upsert behavior — insert or update in one atomic operation.
- Always list column names explicitly for clarity and safety.

## Code Examples

**An upsert with RETURNING — inserts a new product or updates the price if the SKU already exists, then returns the result**

```sql
-- Upsert pattern: insert or update on conflict
INSERT INTO products (sku, name, price)
VALUES ('ABC123', 'Widget', 29.99)
ON CONFLICT (sku)
DO UPDATE SET
    price = EXCLUDED.price,
    updated_at = NOW()
RETURNING id, sku, price;
```


## Resources

- [INSERT Statement](https://www.postgresql.org/docs/18/sql-insert.html) — Complete INSERT syntax reference

---

> 📘 *This lesson is part of the [PostgreSQL Fundamentals](https://stanza.dev/courses/postgresql-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*