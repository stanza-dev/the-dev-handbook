---
source_course: "postgresql-fundamentals"
source_lesson: "postgresql-fundamentals-indexes"
---

# Understanding Indexes

Indexes speed up data retrieval, like an index in a book.

## Why Indexes Matter

Without an index, PostgreSQL must scan every row (Sequential Scan):

```sql
-- Without index: scans 1,000,000 rows to find one
SELECT * FROM users WHERE email = 'alice@example.com';

-- With index: goes directly to the row
```

## Creating Indexes

```sql
-- Basic B-tree index (most common)
CREATE INDEX idx_users_email ON users(email);

-- Unique index (also enforces uniqueness)
CREATE UNIQUE INDEX idx_users_email ON users(email);

-- Index on multiple columns
CREATE INDEX idx_orders_user_date ON orders(user_id, created_at);
```

## When to Create Indexes

✅ **Good candidates:**
- Columns in WHERE clauses
- Columns in JOIN conditions
- Columns in ORDER BY
- Columns with UNIQUE constraint
- Foreign key columns

❌ **Avoid indexing:**
- Small tables (< 1000 rows)
- Columns with few unique values (boolean)
- Columns that change frequently
- Tables with heavy INSERT/UPDATE load

## Index Types

| Type | Best For |
|------|----------|
| B-tree | Most cases, equality and range queries |
| Hash | Equality comparisons only |
| GIN | Arrays, JSONB, full-text search |
| GiST | Geometric data, full-text search |
| BRIN | Very large tables with natural ordering |

## Multi-Column Index Order

Order matters! The index on `(a, b)` is good for:
- `WHERE a = ?`
- `WHERE a = ? AND b = ?`

But NOT for:
- `WHERE b = ?` (can't use the index)

```sql
-- Order columns by selectivity (most selective first)
CREATE INDEX idx_orders ON orders(status, user_id, created_at);
```

## Partial Indexes

Index only some rows:

```sql
-- Only index active users
CREATE INDEX idx_active_users_email 
ON users(email) 
WHERE is_active = true;
```

## Checking Index Usage

```sql
-- See existing indexes
\di

-- See indexes on a table
\d users

-- Check if query uses index
EXPLAIN SELECT * FROM users WHERE email = 'test@example.com';
```

📖 [Indexes Documentation](https://www.postgresql.org/docs/18/indexes.html)

## Resources

- [PostgreSQL Indexes](https://www.postgresql.org/docs/18/indexes.html) — Complete guide to indexes

---

> 📘 *This lesson is part of the [PostgreSQL Fundamentals](https://stanza.dev/courses/postgresql-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*