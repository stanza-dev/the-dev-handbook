---
source_course: "postgresql-fundamentals"
source_lesson: "postgresql-fundamentals-indexes"
---

# Understanding Indexes

## Introduction
Indexes speed up data retrieval by creating efficient lookup structures, similar to an index in a book. Without an index, PostgreSQL must scan every row in the table (a sequential scan) to find matching data. With an index, it can jump directly to the relevant rows.

## Key Concepts
- **Index**: A data structure that speeds up lookups on specific columns. Created with CREATE INDEX.
- **B-tree**: The default and most common index type in PostgreSQL. Efficient for equality (`=`) and range (`<`, `>`, `BETWEEN`) queries.
- **Sequential Scan (Seq Scan)**: Reading every row in the table. Necessary without an index, slow for large tables.
- **Index Scan**: Using an index to jump directly to matching rows. Much faster than sequential scan for selective queries.
- **Partial Index**: An index that covers only rows matching a WHERE condition, saving space and improving performance.

## Real World Context
Indexes are the single most impactful performance optimization in database applications. A query that takes 500ms with a sequential scan on a million-row table can drop to under 1ms with the right index. However, indexes are not free — they slow down writes and consume disk space. Knowing when and where to add indexes is a critical skill.

## Deep Dive

### Why Indexes Matter

The difference between having and not having an index can be dramatic:

```sql
-- Without index: scans 1,000,000 rows to find one
SELECT * FROM users WHERE email = 'alice@example.com';
-- Time: ~500ms

-- With index: jumps directly to the row
CREATE INDEX idx_users_email ON users(email);
-- Time: ~0.1ms
```

The index creates a sorted lookup structure that lets PostgreSQL find rows without reading the entire table.

### Creating Indexes

The basic syntax for creating indexes:

```sql
-- Basic B-tree index (most common)
CREATE INDEX idx_users_email ON users(email);

-- Unique index (also enforces uniqueness)
CREATE UNIQUE INDEX idx_users_email ON users(email);

-- Index on multiple columns
CREATE INDEX idx_orders_user_date ON orders(user_id, created_at);
```

B-tree is the default index type and works for most use cases.

### When to Create Indexes

Not every column needs an index. Here are guidelines:

Good candidates for indexing:
- Columns in WHERE clauses
- Columns in JOIN conditions
- Columns in ORDER BY
- Foreign key columns
- Columns with UNIQUE constraints

Avoid indexing:
- Small tables (fewer than 1000 rows)
- Columns with very few unique values (like booleans)
- Columns that change very frequently
- Tables with heavy INSERT/UPDATE load where read speed is not critical

### Index Types

PostgreSQL offers several index types for different use cases:

| Type | Best For |
|------|----------|
| B-tree | Most cases: equality and range queries |
| Hash | Equality comparisons only |
| GIN | Arrays, JSONB, full-text search |
| GiST | Geometric data, full-text search |
| BRIN | Very large tables with natural ordering |

B-tree handles 90% of use cases. Use specialized types only when B-tree is insufficient.

### Multi-Column Index Order

Column order in multi-column indexes matters. The index on `(a, b)` supports:

```sql
-- Can use the index:
WHERE a = 5
WHERE a = 5 AND b = 10

-- Cannot use the index:
WHERE b = 10  -- leading column 'a' is missing
```

Order columns by selectivity — the most selective column (fewest matching rows) should come first:

```sql
CREATE INDEX idx_orders ON orders(status, user_id, created_at);
```

This index works for queries filtering by status, status + user_id, or status + user_id + created_at.

### Partial Indexes

Index only the rows you actually query:

```sql
-- Only index active users (saves space, faster writes)
CREATE INDEX idx_active_users_email 
ON users(email) 
WHERE is_active = true;
```

Partial indexes are smaller and faster to maintain because they only cover a subset of rows.

### Checking Index Usage

Verify that your indexes are being used:

```sql
-- See existing indexes on a table
\d users

-- Check if a query uses an index
EXPLAIN SELECT * FROM users WHERE email = 'test@example.com';
```

The EXPLAIN output shows whether PostgreSQL chose an Index Scan or a Seq Scan.

## Common Pitfalls
1. **Over-indexing** — Every index slows down INSERT, UPDATE, and DELETE because the index must be updated too. Only create indexes that your queries actually use.
2. **Wrong column order in multi-column indexes** — An index on `(a, b)` cannot be used for `WHERE b = 10` alone. The leading column must be in the query.
3. **Not indexing foreign keys** — PostgreSQL does not automatically index foreign key columns. Always add indexes on FK columns to speed up JOINs.

## Best Practices
1. **Index foreign key columns** — Every foreign key should have an index for JOIN performance.
2. **Use partial indexes for filtered queries** — If you always query `WHERE is_active = true`, a partial index on that condition is smaller and faster.
3. **Use EXPLAIN to verify** — After creating an index, use EXPLAIN to confirm PostgreSQL actually uses it for your queries.

## Summary
- Indexes create efficient lookup structures that speed up SELECT queries.
- B-tree is the default and most common index type.
- Multi-column index order matters — the leading column must appear in the query.
- Partial indexes cover only matching rows, saving space and maintenance cost.
- Every index slows down writes — only create indexes that your queries need.
- Always index foreign key columns and use EXPLAIN to verify index usage.

## Code Examples

**Three indexing strategies: unique index for lookups, multi-column index with sort order, and partial index for filtered queries**

```sql
-- Create targeted indexes based on query patterns
-- For: WHERE email = 'x' (login lookup)
CREATE UNIQUE INDEX idx_users_email ON users(email);

-- For: WHERE user_id = x ORDER BY created_at DESC (user's orders)
CREATE INDEX idx_orders_user_date ON orders(user_id, created_at DESC);

-- Partial index for active records only
CREATE INDEX idx_active_products ON products(category_id)
WHERE is_active = true;
```


## Resources

- [PostgreSQL Indexes](https://www.postgresql.org/docs/18/indexes.html) — Complete guide to indexes

---

> 📘 *This lesson is part of the [PostgreSQL Fundamentals](https://stanza.dev/courses/postgresql-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*