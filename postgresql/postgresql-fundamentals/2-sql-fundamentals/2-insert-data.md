---
source_course: "postgresql-fundamentals"
source_lesson: "postgresql-fundamentals-insert-data"
---

# Inserting Data with INSERT

The `INSERT` statement adds new rows to a table.

## Basic INSERT Syntax

```sql
-- Insert a single row
INSERT INTO books (title, author, published_year, pages)
VALUES ('The Pragmatic Programmer', 'David Thomas', 2019, 352);

-- Insert with all columns (must match table order)
INSERT INTO books
VALUES (DEFAULT, '1984', 'George Orwell', 1949, 328, DEFAULT);
```

## Inserting Multiple Rows

```sql
INSERT INTO books (title, author, published_year, pages)
VALUES 
    ('Clean Code', 'Robert C. Martin', 2008, 464),
    ('Design Patterns', 'Gang of Four', 1994, 395),
    ('Refactoring', 'Martin Fowler', 2018, 448);
```

## INSERT with RETURNING

Get back the inserted data (great for getting auto-generated IDs):

```sql
INSERT INTO books (title, author, published_year)
VALUES ('PostgreSQL Up and Running', 'Regina Obe', 2022)
RETURNING id, title, created_at;
```

Output:
```
 id |           title            |         created_at
----+----------------------------+----------------------------
  5 | PostgreSQL Up and Running  | 2024-01-15 10:30:00.123456
```

## INSERT with Default Values

```sql
-- Use DEFAULT for specific columns
INSERT INTO books (title, author, pages)
VALUES ('The Art of SQL', 'Stephane Faroult', DEFAULT);

-- Insert a row with all defaults
INSERT INTO logs DEFAULT VALUES;
```

## INSERT from SELECT (Copying Data)

```sql
-- Copy data from another table
INSERT INTO archived_books (title, author, published_year)
SELECT title, author, published_year
FROM books
WHERE published_year < 2000;
```

## Handling Conflicts (UPSERT)

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

📖 [INSERT Documentation](https://www.postgresql.org/docs/18/sql-insert.html)

## Resources

- [INSERT Statement](https://www.postgresql.org/docs/18/sql-insert.html) — Complete INSERT syntax reference

---

> 📘 *This lesson is part of the [PostgreSQL Fundamentals](https://stanza.dev/courses/postgresql-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*