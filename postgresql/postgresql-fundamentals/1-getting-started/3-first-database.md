---
source_course: "postgresql-fundamentals"
source_lesson: "postgresql-fundamentals-first-database"
---

# Creating Your First Database

## Introduction
A PostgreSQL server can host multiple databases, each acting as an isolated container for your tables, views, and other objects. In this lesson, you will create your first database, learn naming conventions, and build your first table using modern PostgreSQL syntax.

## Key Concepts
- **Database**: An isolated container within a PostgreSQL server that holds tables, views, indexes, and other objects.
- **Table**: A structured collection of rows and columns where data is stored.
- **IDENTITY column**: The modern way (PostgreSQL 10+) to create auto-incrementing integer columns, replacing the legacy SERIAL approach.
- **PRIMARY KEY**: A constraint that uniquely identifies each row in a table.
- **NOT NULL**: A constraint that prevents a column from storing empty (NULL) values.

## Real World Context
Every application needs a database. Whether you are building a blog, an e-commerce site, or an API, the first step is always `CREATE DATABASE` followed by `CREATE TABLE`. Getting the schema right from the start saves significant refactoring time later.

## Deep Dive

### Creating a Database

You can create a database from within psql or from the command line. Here is how to do it from psql:

```sql
-- Create a new database
CREATE DATABASE learning_postgres;

-- List all databases
\l

-- Connect to your new database
\c learning_postgres
```

The `\l` command shows all databases on the server, and `\c` switches your connection to the specified database.

From the command line, you can also use the `createdb` utility:

```bash
# Create database
createdb learning_postgres

# Connect directly
psql learning_postgres
```

This is equivalent to the SQL `CREATE DATABASE` command but runs outside psql.

### Database Naming Conventions

- Use lowercase letters
- Separate words with underscores: `my_app_development`
- Avoid reserved keywords
- Keep names descriptive but concise

### Your First Table

Let's create a simple table to store books. Notice we use `GENERATED ALWAYS AS IDENTITY` instead of the older `SERIAL` approach:

```sql
-- Modern approach (PostgreSQL 10+, recommended)
CREATE TABLE books (
    id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    author VARCHAR(100) NOT NULL,
    published_year INTEGER,
    pages INTEGER,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
-- Legacy approach (still works):
-- id SERIAL PRIMARY KEY
```

Here is what each part means:
- `GENERATED ALWAYS AS IDENTITY`: Auto-incrementing integer — the modern SQL-standard way to create auto-incrementing columns, replacing the PostgreSQL-specific `SERIAL` type
- `PRIMARY KEY`: Unique identifier for each row
- `VARCHAR(n)`: Variable-length text up to n characters
- `NOT NULL`: This column cannot be empty
- `INTEGER`: Whole number
- `TIMESTAMP`: Date and time value
- `DEFAULT`: Value used when none is provided

### Verifying Your Table

After creating the table, verify its structure with psql commands:

```sql
-- List tables
\dt

-- Describe table structure
\d books
```

The `\d books` command shows the columns, types, and constraints of your table.

## Common Pitfalls
1. **Using SERIAL instead of IDENTITY** — `SERIAL` still works but is a PostgreSQL-specific legacy feature. `GENERATED ALWAYS AS IDENTITY` follows the SQL standard and gives you more control (e.g., preventing manual ID insertion).
2. **Forgetting NOT NULL on required columns** — Without `NOT NULL`, columns accept empty values by default. Always add `NOT NULL` to columns that must have data.
3. **Not connecting to the right database** — If you create a table without first running `\c learning_postgres`, it ends up in the default `postgres` database.

## Best Practices
1. **Use IDENTITY columns for auto-incrementing IDs** — They follow the SQL standard and prevent accidental manual ID insertion with `GENERATED ALWAYS`.
2. **Always define a PRIMARY KEY** — Every table should have a primary key to uniquely identify rows. This is essential for joins, updates, and deletes.
3. **Use descriptive column names** — Names like `published_year` are clearer than `pub_yr` and make queries self-documenting.

## Summary
- Create databases with `CREATE DATABASE` and connect with `\c`.
- Use `GENERATED ALWAYS AS IDENTITY` for auto-incrementing columns (modern, SQL-standard approach).
- `SERIAL` is the legacy approach — it still works but IDENTITY is recommended.
- Every table should have a `PRIMARY KEY` and use `NOT NULL` on required columns.
- Verify your table structure with `\d tablename`.

## Code Examples

**Creating a table with IDENTITY (the modern SQL-standard auto-increment) and inserting a row**

```sql
-- Create a table with IDENTITY column (modern approach)
CREATE TABLE books (
    id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    author VARCHAR(100) NOT NULL,
    published_year INTEGER,
    pages INTEGER,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Insert data (id is auto-generated)
INSERT INTO books (title, author, published_year, pages)
VALUES ('The Pragmatic Programmer', 'David Thomas', 2019, 352)
RETURNING id, title;
```

**SERIAL vs IDENTITY — IDENTITY prevents accidental manual ID insertion, making it safer**

```sql
-- SERIAL (legacy) vs IDENTITY (modern) comparison
-- Legacy:
-- id SERIAL PRIMARY KEY
-- Modern:
-- id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY

-- IDENTITY prevents manual ID insertion (safer):
-- INSERT INTO books (id, title, ...) VALUES (99, ...)
-- ERROR: cannot insert a non-DEFAULT value into column "id"

-- If you need to override (rare, e.g. migrations):
-- INSERT INTO books OVERRIDING SYSTEM VALUE (id, title, ...)
-- VALUES (99, ...);
```


## Resources

- [CREATE TABLE Documentation](https://www.postgresql.org/docs/18/sql-createtable.html) — Complete reference for CREATE TABLE syntax

---

> 📘 *This lesson is part of the [PostgreSQL Fundamentals](https://stanza.dev/courses/postgresql-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*