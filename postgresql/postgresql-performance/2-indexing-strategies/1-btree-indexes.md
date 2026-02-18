---
source_course: "postgresql-performance"
source_lesson: "postgresql-performance-btree-indexes"
---

# B-tree Indexes

B-tree is the default and most versatile index type in PostgreSQL.

## B-tree Structure

```
            ┌─────────────────┐
            │  Root Node      │
            │  [50, 100, 150] │
            └───────┬─────────┘
        ┌───────────┼───────────┐
        ▼           ▼           ▼
   ┌─────────┐ ┌─────────┐ ┌─────────┐
   │ Internal│ │ Internal│ │ Internal│
   │[10,25,40]│ │[60,75,90]│ │[110,125]│
   └────┬────┘ └────┬────┘ └────┬────┘
        ▼           ▼           ▼
    [Leaf Pages with Row Pointers]
```

**Properties:**
- Balanced tree (all leaves at same depth)
- O(log n) lookup time
- Sorted order enables range queries

## Supported Operators

B-tree indexes support:
- Equality: `=`
- Range: `<`, `<=`, `>`, `>=`, `BETWEEN`
- Pattern matching: `LIKE 'prefix%'` (prefix only)
- Sorting: `ORDER BY` (if index matches)
- `IS NULL` / `IS NOT NULL`

```sql
-- All these queries can use a B-tree index on email
SELECT * FROM users WHERE email = 'alice@example.com';
SELECT * FROM users WHERE email > 'a' AND email < 'b';
SELECT * FROM users WHERE email LIKE 'alice%';
SELECT * FROM users ORDER BY email;
```

## Creating B-tree Indexes

```sql
-- Simple index
CREATE INDEX idx_users_email ON users(email);

-- Unique index
CREATE UNIQUE INDEX idx_users_email ON users(email);

-- Descending order
CREATE INDEX idx_orders_date ON orders(created_at DESC);

-- Control NULL positioning
CREATE INDEX idx_orders_priority 
ON orders(priority NULLS FIRST);
```

## Index Selectivity

**Selectivity** = unique values / total rows

```sql
-- Check selectivity
SELECT 
    COUNT(DISTINCT email)::float / COUNT(*) AS email_selectivity,
    COUNT(DISTINCT status)::float / COUNT(*) AS status_selectivity
FROM orders;
```

Results:
```
 email_selectivity | status_selectivity
-------------------+--------------------
            0.95   |              0.001
```

- **email**: High selectivity (95% unique) → Good index candidate
- **status**: Low selectivity (few distinct values) → Poor index candidate

## Multi-Column Index Order

Order matters! Index on `(a, b, c)` can be used for:

✅ `WHERE a = ?`
✅ `WHERE a = ? AND b = ?`
✅ `WHERE a = ? AND b = ? AND c = ?`
✅ `WHERE a = ? ORDER BY b`

❌ `WHERE b = ?` (can't skip leading column)
❌ `WHERE c = ?`

```sql
-- Multi-column index
CREATE INDEX idx_orders_user_date 
ON orders(user_id, created_at DESC);

-- Efficient for:
SELECT * FROM orders WHERE user_id = 42 ORDER BY created_at DESC;
```

📖 [B-Tree Indexes](https://www.postgresql.org/docs/18/indexes-types.html)

## Resources

- [Index Types](https://www.postgresql.org/docs/18/indexes-types.html) — All PostgreSQL index types

---

> 📘 *This lesson is part of the [PostgreSQL Performance Engineering](https://stanza.dev/courses/postgresql-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*