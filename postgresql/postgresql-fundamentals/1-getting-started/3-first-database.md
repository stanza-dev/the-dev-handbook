---
source_course: "postgresql-fundamentals"
source_lesson: "postgresql-fundamentals-first-database"
---

# Creating Your First Database

A PostgreSQL server can host multiple databases. Each database is an isolated container for your tables, views, and other objects.

## Creating a Database

From psql:

```sql
-- Create a new database
CREATE DATABASE learning_postgres;

-- List all databases
\l

-- Connect to your new database
\c learning_postgres
```

From the command line:

```bash
# Create database
createdb learning_postgres

# Connect directly
psql learning_postgres
```

## Database Naming Conventions

- Use lowercase letters
- Separate words with underscores: `my_app_development`
- Avoid reserved keywords
- Keep names descriptive but concise

## Your First Table

Let's create a simple table to store books:

```sql
CREATE TABLE books (
    id SERIAL PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    author VARCHAR(100) NOT NULL,
    published_year INTEGER,
    pages INTEGER,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**Understanding the syntax:**
- `SERIAL`: Auto-incrementing integer (creates a sequence automatically)
- `PRIMARY KEY`: Unique identifier for each row
- `VARCHAR(n)`: Variable-length text up to n characters
- `NOT NULL`: This column cannot be empty
- `INTEGER`: Whole number
- `TIMESTAMP`: Date and time value
- `DEFAULT`: Value used when none is provided

## Verifying Your Table

```sql
-- List tables
\dt

-- Describe table structure
\d books
```

Output:
```
                                     Table "public.books"
    Column     |            Type             | Nullable |              Default
---------------+-----------------------------+----------+------------------------------------
 id            | integer                     | not null | nextval('books_id_seq'::regclass)
 title         | character varying(255)      | not null |
 author        | character varying(100)      | not null |
 published_year| integer                     |          |
 pages         | integer                     |          |
 created_at    | timestamp without time zone |          | CURRENT_TIMESTAMP
```

📖 [CREATE DATABASE Documentation](https://www.postgresql.org/docs/18/sql-createdatabase.html)

## Resources

- [CREATE TABLE Documentation](https://www.postgresql.org/docs/18/sql-createtable.html) — Complete reference for CREATE TABLE syntax

---

> 📘 *This lesson is part of the [PostgreSQL Fundamentals](https://stanza.dev/courses/postgresql-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*