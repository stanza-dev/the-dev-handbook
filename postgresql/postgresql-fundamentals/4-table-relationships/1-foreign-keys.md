---
source_course: "postgresql-fundamentals"
source_lesson: "postgresql-fundamentals-foreign-keys"
---

# Primary and Foreign Keys

Relational databases connect tables through keys. Understanding keys is fundamental to database design.

## Primary Keys

A PRIMARY KEY uniquely identifies each row:

```sql
-- Single column primary key
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
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

## Foreign Keys

FOREIGN KEYs create relationships between tables:

```sql
-- Authors table
CREATE TABLE authors (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    country VARCHAR(50)
);

-- Books table with foreign key to authors
CREATE TABLE books (
    id SERIAL PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    author_id INTEGER NOT NULL REFERENCES authors(id),
    published_year INTEGER
);
```

## Foreign Key Constraints

Control what happens when referenced data changes:

```sql
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL,
    total NUMERIC(10, 2),
    created_at TIMESTAMPTZ DEFAULT NOW(),
    
    CONSTRAINT fk_user
        FOREIGN KEY (user_id)
        REFERENCES users(id)
        ON DELETE CASCADE      -- Delete orders when user is deleted
        ON UPDATE CASCADE      -- Update foreign key when user id changes
);
```

### ON DELETE Options:

| Option | Behavior |
|--------|----------|
| `CASCADE` | Delete related rows |
| `SET NULL` | Set foreign key to NULL |
| `SET DEFAULT` | Set to default value |
| `RESTRICT` | Prevent delete if references exist |
| `NO ACTION` | Like RESTRICT, but checked at end of transaction |

## Example: One-to-Many Relationship

```sql
-- One author has many books
CREATE TABLE authors (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL
);

CREATE TABLE books (
    id SERIAL PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    author_id INTEGER REFERENCES authors(id)
);

-- Insert data
INSERT INTO authors (name) VALUES ('George Orwell');
INSERT INTO books (title, author_id) VALUES ('1984', 1);
INSERT INTO books (title, author_id) VALUES ('Animal Farm', 1);
```

## Example: Many-to-Many Relationship

```sql
-- Books can have many tags, tags can be on many books
CREATE TABLE tags (
    id SERIAL PRIMARY KEY,
    name VARCHAR(50) UNIQUE NOT NULL
);

-- Junction table for many-to-many
CREATE TABLE book_tags (
    book_id INTEGER REFERENCES books(id) ON DELETE CASCADE,
    tag_id INTEGER REFERENCES tags(id) ON DELETE CASCADE,
    PRIMARY KEY (book_id, tag_id)
);
```

📖 [Constraints Documentation](https://www.postgresql.org/docs/18/ddl-constraints.html)

## Resources

- [Constraints](https://www.postgresql.org/docs/18/ddl-constraints.html) — Complete guide to table constraints

---

> 📘 *This lesson is part of the [PostgreSQL Fundamentals](https://stanza.dev/courses/postgresql-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*