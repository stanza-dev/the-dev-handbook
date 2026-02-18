---
source_course: "postgresql-replication"
source_lesson: "publications-basics"
---

# Creating Publications

Publications define what data is replicated in logical replication. They are created on the publisher (source) server and specify which tables and operations to replicate.

## Requirements

```sql
-- Requires wal_level = logical
SHOW wal_level;  -- Must be 'logical'

-- Tables must have REPLICA IDENTITY
-- (defaults to PRIMARY KEY)
```

## Creating Publications

```sql
-- Publish all tables in database
CREATE PUBLICATION pub_all FOR ALL TABLES;

-- Publish specific tables
CREATE PUBLICATION pub_orders FOR TABLE orders, order_items;

-- Publish all tables in schema
CREATE PUBLICATION pub_sales FOR TABLES IN SCHEMA sales;

-- Add table to existing publication
ALTER PUBLICATION pub_orders ADD TABLE products;

-- Remove table
ALTER PUBLICATION pub_orders DROP TABLE order_items;
```

## Controlling Published Operations

```sql
-- Only publish INSERTs (no UPDATE/DELETE)
CREATE PUBLICATION pub_insert_only FOR TABLE audit_log
    WITH (publish = 'insert');

-- Publish INSERT, UPDATE, DELETE (exclude TRUNCATE)
CREATE PUBLICATION pub_no_truncate FOR TABLE users
    WITH (publish = 'insert, update, delete');

-- Available operations: insert, update, delete, truncate
```

## Viewing Publications

```sql
-- List all publications
SELECT * FROM pg_publication;

-- See which tables are in a publication
SELECT * FROM pg_publication_tables
WHERE pubname = 'pub_orders';
```

## Code Examples

```undefined
-- Create a publication for specific tables
CREATE PUBLICATION orders_pub FOR TABLE orders, order_items;

-- Create publication with row filter
CREATE PUBLICATION active_users_pub 
    FOR TABLE users WHERE (active = true);

-- View publication details
SELECT pubname, puballtables, pubinsert, pubupdate, pubdelete, pubtruncate
FROM pg_publication;
```


## Resources

- [CREATE PUBLICATION](https://www.postgresql.org/docs/current/sql-createpublication.html) — undefined

---

> 📘 *This lesson is part of the [High Availability & Replication in PostgreSQL](https://stanza.dev/courses/postgresql-replication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*