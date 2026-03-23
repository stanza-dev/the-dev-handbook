---
source_course: "postgresql-replication"
source_lesson: "postgresql-replication-publications"
---

# Creating Publications

## Introduction

Publications define what data is replicated in logical replication. They are created on the publisher (source) server and specify which tables and operations to replicate.

This lesson covers publication creation, operation filtering, and the new PostgreSQL 18 option for replicating generated columns.

## Key Concepts

- **Publication**: A named set of tables and operations to replicate from the publisher server.
- **publish parameter**: Controls which DML operations (insert, update, delete, truncate) are included in the publication.
- **REPLICA IDENTITY**: Determines which columns are included in UPDATE and DELETE WAL records for logical decoding. Defaults to the primary key.
- **publish_generated_columns (PG18)**: A new option controlling whether stored generated columns are included in replicated data.

## Real World Context

An e-commerce platform needs to feed its analytics warehouse with only order data, excluding customer passwords and other sensitive tables. A publication with specific table selection and column filtering lets the analytics team access the data they need without exposing sensitive information. With PostgreSQL 18, they can now also include computed columns like `total_with_tax` that are stored generated columns.

## Deep Dive

Logical replication requires `wal_level = logical`:

```sql
-- Verify wal_level
SHOW wal_level;  -- Must be 'logical'
```

Create publications with various scoping options:

```sql
-- Publish all tables in database
CREATE PUBLICATION pub_all FOR ALL TABLES;

-- Publish specific tables
CREATE PUBLICATION pub_orders FOR TABLE orders, order_items;

-- Publish all tables in a schema
CREATE PUBLICATION pub_sales FOR TABLES IN SCHEMA sales;

-- Add or remove tables from existing publication
ALTER PUBLICATION pub_orders ADD TABLE products;
ALTER PUBLICATION pub_orders DROP TABLE order_items;
```

Each approach offers different granularity: all tables is convenient but replicates everything, specific tables give precise control, and schema-based captures new tables automatically.

Control which operations are published:

```sql
-- Only publish INSERTs (no UPDATE/DELETE)
CREATE PUBLICATION pub_insert_only FOR TABLE audit_log
    WITH (publish = 'insert');

-- Publish INSERT, UPDATE, DELETE (exclude TRUNCATE)
CREATE PUBLICATION pub_no_truncate FOR TABLE users
    WITH (publish = 'insert, update, delete');
```

The `publish` parameter accepts any combination of: insert, update, delete, truncate.

### Generated Column Replication (PostgreSQL 18)

PostgreSQL 18 adds the `publish_generated_columns` option:

```sql
-- Replicate stored generated columns
CREATE PUBLICATION pub_with_generated FOR TABLE orders
    WITH (publish_generated_columns = 'stored');

-- Default is 'none' (generated columns not replicated)
CREATE PUBLICATION pub_default FOR TABLE orders
    WITH (publish_generated_columns = 'none');
```

There are important constraints to understand:

- Only **stored** generated columns can be replicated; virtual generated columns cannot.
- On the subscriber side, the corresponding column must be a **regular** (non-generated) column, because the subscriber receives the value directly rather than computing it.
- This is useful when the subscriber does not have the functions or data needed to compute the generated value.

View publication details:

```sql
-- List all publications
SELECT * FROM pg_publication;

-- See which tables are in a publication
SELECT * FROM pg_publication_tables
WHERE pubname = 'pub_orders';
```

These catalog queries help verify that the publication includes the expected tables and configuration.

## Common Pitfalls

- **FOR ALL TABLES capturing system or temporary tables**: This publishes every table in the database, including ones you may not intend. Use schema-based or explicit table lists for production.
- **Forgetting REPLICA IDENTITY for UPDATE/DELETE**: Without an appropriate replica identity, UPDATE and DELETE operations fail during logical decoding. Set it to the primary key or a unique index.
- **Assuming virtual generated columns replicate**: Only stored generated columns support replication; virtual generated columns cannot be published.

## Best Practices

- Use explicit table lists or schema-based publication for production systems rather than FOR ALL TABLES.
- Always verify REPLICA IDENTITY is set for tables that will receive UPDATE or DELETE operations.
- On PostgreSQL 18+, use `publish_generated_columns = 'stored'` when subscribers need computed values without the ability to regenerate them.

## Summary

- Publications define which tables and operations are replicated from the publisher.
- Scope publications to specific tables, schemas, or all tables depending on your needs.
- The `publish` parameter controls which DML operations are included.
- PostgreSQL 18 adds `publish_generated_columns` to replicate stored generated column values.
- Subscribers must receive generated column values as regular columns, not as generated columns.

## Code Examples

**Publication with Filtering and Generated Columns**

```sql
-- Create a publication for specific tables
CREATE PUBLICATION orders_pub FOR TABLE orders, order_items;

-- Create publication with row filter
CREATE PUBLICATION active_users_pub
    FOR TABLE users WHERE (active = true);

-- PG18: Include stored generated columns
CREATE PUBLICATION full_orders_pub FOR TABLE orders
    WITH (publish_generated_columns = 'stored');

-- View publication details
SELECT pubname, puballtables, pubinsert, pubupdate, pubdelete, pubtruncate
FROM pg_publication;
```


## Resources

- [CREATE PUBLICATION](https://www.postgresql.org/docs/current/sql-createpublication.html) — undefined

---

> 📘 *This lesson is part of the [High Availability & Replication in PostgreSQL](https://stanza.dev/courses/postgresql-replication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*