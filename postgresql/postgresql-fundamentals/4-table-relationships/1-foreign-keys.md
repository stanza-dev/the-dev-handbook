---
source_course: "postgresql-fundamentals"
source_lesson: "postgresql-fundamentals-foreign-keys"
---

# Primary and Foreign Keys

## Introduction
Relational databases connect tables through keys. PRIMARY KEY uniquely identifies each row, and FOREIGN KEY creates links between tables. Understanding keys is fundamental to database design — without them, your data has no structure or integrity.

## Key Concepts
- **PRIMARY KEY**: A constraint that uniquely identifies each row in a table. No two rows can have the same primary key value, and it cannot be NULL.
- **FOREIGN KEY**: A column (or set of columns) that references a primary key in another table, creating a relationship between the two tables.
- **Referential Integrity**: The guarantee that every foreign key value actually points to an existing row in the referenced table.
- **ON DELETE CASCADE**: A foreign key option that automatically deletes child rows when the parent row is deleted.
- **Composite Primary Key**: A primary key made up of two or more columns, where the combination must be unique.

## Real World Context
Every database-backed application uses foreign keys: users have orders, orders have items, items belong to products. Without foreign keys, you could have orders pointing to non-existent users, creating orphaned data that breaks your application. Referential integrity prevents this class of bugs entirely.

## Deep Dive

### Primary Keys

A PRIMARY KEY uniquely identifies each row and prevents duplicates:

```sql
-- Single column primary key
CREATE TABLE users (
    id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    email VARCHAR(255) NOT NULL UNIQUE,
    name VARCHAR(100) NOT NULL
);

-- Composite primary key
CREATE TABLE order_items (
    order_id INTEGER NOT NULL,
    product_id INTEGER NOT NULL,
    quantity INTEGER NOT NULL,
    PRIMARY KEY (order_id, product_id)
);
```

Composite primary keys are common in junction tables (many-to-many relationships) where the combination of two foreign keys is unique.

### Foreign Keys

FOREIGN KEYs create relationships between tables and enforce referential integrity:

```sql
-- Authors table
CREATE TABLE authors (
    id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    country VARCHAR(50)
);

-- Books table with foreign key to authors
CREATE TABLE books (
    id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    author_id INTEGER NOT NULL REFERENCES authors(id),
    published_year INTEGER
);
```

The `REFERENCES authors(id)` clause ensures that every `author_id` in the books table corresponds to an existing author.

### Foreign Key Constraints

You can control what happens when referenced data changes:

```sql
CREATE TABLE orders (
    id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    user_id INTEGER NOT NULL,
    total NUMERIC(10, 2),
    created_at TIMESTAMPTZ DEFAULT NOW(),
    
    CONSTRAINT fk_user
        FOREIGN KEY (user_id)
        REFERENCES users(id)
        ON DELETE CASCADE
        ON UPDATE CASCADE
);
```

The ON DELETE and ON UPDATE options determine what happens to child rows when the parent changes:

| Option | Behavior |
|--------|----------|
| `CASCADE` | Delete/update related rows |
| `SET NULL` | Set foreign key to NULL |
| `SET DEFAULT` | Set to default value |
| `RESTRICT` | Prevent delete if references exist |
| `NO ACTION` | Like RESTRICT, but checked at end of transaction |

CASCADE is common for tightly coupled data (delete user -> delete their orders). RESTRICT is safer when you want to prevent accidental deletions.

### One-to-Many Relationship

The most common relationship pattern — one author has many books:

```sql
CREATE TABLE authors (
    id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name VARCHAR(100) NOT NULL
);

CREATE TABLE books (
    id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    author_id INTEGER REFERENCES authors(id)
);

-- Insert data
INSERT INTO authors (name) VALUES ('George Orwell');
INSERT INTO books (title, author_id) VALUES ('1984', 1);
INSERT INTO books (title, author_id) VALUES ('Animal Farm', 1);
```

The foreign key `author_id` on the books table creates the one-to-many relationship.

### Many-to-Many Relationship

Many-to-many relationships require a junction table:

```sql
-- Books can have many tags, tags can be on many books
CREATE TABLE tags (
    id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name VARCHAR(50) UNIQUE NOT NULL
);

-- Junction table for many-to-many
CREATE TABLE book_tags (
    book_id INTEGER REFERENCES books(id) ON DELETE CASCADE,
    tag_id INTEGER REFERENCES tags(id) ON DELETE CASCADE,
    PRIMARY KEY (book_id, tag_id)
);
```

The `book_tags` junction table has a composite primary key and two foreign keys, creating the many-to-many relationship.

## Common Pitfalls
1. **Forgetting to index foreign key columns** — PostgreSQL does not automatically create indexes on foreign key columns (unlike the primary key). Add indexes on foreign keys for better JOIN performance.
2. **Using CASCADE without thinking** — ON DELETE CASCADE can silently delete large amounts of related data. Use RESTRICT as the default and only use CASCADE when you are sure.

## Best Practices
1. **Always define foreign keys explicitly** — Even if your application enforces relationships in code, database-level foreign keys prevent bugs and provide documentation.
2. **Index foreign key columns** — Create an index on every foreign key column to speed up JOINs and CASCADE operations.
3. **Default to RESTRICT for ON DELETE** — It is safer to prevent accidental deletions than to cascade them. Switch to CASCADE only when the business logic requires it.

## Summary
- PRIMARY KEY uniquely identifies rows; FOREIGN KEY creates relationships between tables.
- Referential integrity ensures foreign key values always point to existing rows.
- ON DELETE CASCADE deletes child rows when the parent is deleted; RESTRICT prevents it.
- One-to-many relationships use a foreign key on the "many" side.
- Many-to-many relationships require a junction table with composite primary key.
- Always index foreign key columns for performance.

## Code Examples

**Setting up a one-to-many relationship with IDENTITY columns, a foreign key, and an index for performance**

```sql
-- Complete one-to-many relationship with IDENTITY and FK
CREATE TABLE authors (
    id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name VARCHAR(100) NOT NULL
);

CREATE TABLE books (
    id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    author_id INTEGER NOT NULL REFERENCES authors(id) ON DELETE RESTRICT
);

-- Index the foreign key for JOIN performance
CREATE INDEX idx_books_author_id ON books(author_id);
```


## Resources

- [Constraints](https://www.postgresql.org/docs/18/ddl-constraints.html) — Complete guide to table constraints

---

> 📘 *This lesson is part of the [PostgreSQL Fundamentals](https://stanza.dev/courses/postgresql-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*