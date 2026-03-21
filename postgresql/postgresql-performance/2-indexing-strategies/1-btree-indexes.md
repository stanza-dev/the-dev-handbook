---
source_course: "postgresql-performance"
source_lesson: "postgresql-performance-btree-indexes"
---

# B-tree Indexes

## Introduction

B-tree is the default and most versatile index type in PostgreSQL. It handles equality checks, range queries, sorting, and even pattern matching. Nearly every PostgreSQL database relies heavily on B-tree indexes, making them the first index type you should master.

## Key Concepts

- **B-tree**: A balanced tree data structure that maintains sorted data and allows O(log n) lookups.
- **Selectivity**: The ratio of distinct values to total rows - higher selectivity means the index is more useful.
- **Multi-column index**: An index on multiple columns where column order determines which queries can use it.
- **Covering index**: An index with INCLUDE columns that enables Index Only Scans by including all columns a query needs.
- **Skip scan**: A PostgreSQL 18 feature allowing multicolumn indexes to be used even without a filter on the leading column.

## Real World Context

B-tree indexes are behind every primary key, every unique constraint, and most WHERE clauses in production databases. Choosing the right column order for a multi-column B-tree index can mean the difference between a 2ms query and a 2-second query. Understanding selectivity prevents you from creating useless indexes that slow down writes.

## Deep Dive

### B-tree Structure

A B-tree is a balanced tree where all leaf nodes are at the same depth:

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

This structure gives O(log n) lookup time and keeps data sorted for range queries.

### Supported Operators

B-tree indexes support equality, range, prefix pattern matching, sorting, and NULL checks:

```sql
-- All these queries can use a B-tree index on email
SELECT * FROM users WHERE email = 'alice@example.com';
SELECT * FROM users WHERE email > 'a' AND email < 'b';
SELECT * FROM users WHERE email LIKE 'alice%';
SELECT * FROM users ORDER BY email;
```

These operators work because B-tree data is sorted, enabling efficient traversal in any direction.

### Creating B-tree Indexes

Here are the common forms of B-tree index creation:

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

Each variant serves a different query pattern. DESC indexes are useful when your common query sorts in descending order.

### Index Selectivity

Selectivity determines whether an index is worth using. You can check it with a simple query:

```sql
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

Email has 95% selectivity (good index candidate), while status has only 0.1% selectivity (poor candidate for standalone B-tree index).

### Multi-Column Index Order

Column order in a multi-column index determines which queries can use it. An index on `(a, b, c)` supports these query patterns:

- `WHERE a = ?` - uses the leading column
- `WHERE a = ? AND b = ?` - uses two leading columns
- `WHERE a = ? AND b = ? AND c = ?` - uses all three columns
- `WHERE a = ? ORDER BY b` - uses the index for both filter and sort

But it cannot efficiently serve `WHERE b = ?` or `WHERE c = ?` because these skip the leading column.

```sql
CREATE INDEX idx_orders_user_date 
ON orders(user_id, created_at DESC);

-- Efficient: uses both columns
SELECT * FROM orders WHERE user_id = 42 ORDER BY created_at DESC;
```

This index is optimized for retrieving a specific user's most recent orders.

### PostgreSQL 18: Skip Scan

PostgreSQL 18 introduces skip scan for multicolumn B-tree indexes, allowing them to be useful even when the query lacks a restriction on the leading column or uses a non-equality condition on it:

```sql
-- Index on (status, created_at)
CREATE INDEX idx_orders_status_date ON orders(status, created_at);

-- Previously this query could NOT use the index (no filter on 'status').
-- With PG18 skip scan, it iterates through each distinct 'status' value
-- and performs a range lookup on created_at within each group:
SELECT * FROM orders WHERE created_at > '2024-01-01';
```

Skip scan works by internally iterating through the distinct values of the leading column(s) and performing a separate index lookup for each one. It is most effective when the leading column has few distinct values (e.g., a status column with 3-5 values). If the leading column has high cardinality, the planner will prefer a sequential scan instead.

## Common Pitfalls

1. **Creating indexes on low-selectivity columns** - A B-tree index on a boolean column or a status column with three values is rarely useful. The planner will prefer a sequential scan.
2. **Wrong column order in multi-column indexes** - Putting the less selective column first means the index cannot efficiently filter by the more selective column alone.
3. **Over-indexing** - Each index slows down INSERT, UPDATE, and DELETE operations. Only create indexes for actual query patterns in your workload.

## Best Practices

1. **Put the most selective column first** - In a multi-column index, the leading column should have the highest selectivity for your most common queries.
2. **Use INCLUDE for covering indexes** - Add frequently selected columns with INCLUDE to enable Index Only Scans without bloating the index key.
3. **Monitor index usage** - Check `pg_stat_user_indexes.idx_scan` to find unused indexes that can be dropped to reduce write overhead.

## Summary

- B-tree is the default index type, supporting equality, range, sorting, and prefix matching.
- Selectivity determines whether an index is useful - aim for high selectivity columns.
- Multi-column index column order determines which queries benefit.
- PostgreSQL 18 skip scan allows using multicolumn indexes without filtering on the leading column.
- Every index has a write cost - only index columns that serve real query patterns.

## Code Examples

**Checking selectivity to decide whether to index a column, and creating a covering index with INCLUDE for Index Only Scans**

```sql
-- Check index selectivity before creating an index
SELECT 
    COUNT(DISTINCT status)::float / COUNT(*) AS selectivity
FROM orders;
-- Result: 0.001 → poor candidate (too few distinct values)

-- Multi-column index with covering columns
CREATE INDEX idx_orders_covering
ON orders(user_id, created_at DESC)
INCLUDE (total, status);

-- Enables Index Only Scan for this common query:
SELECT total, status FROM orders
WHERE user_id = 42 ORDER BY created_at DESC;
```

**PostgreSQL 18 skip scan iterates through distinct leading column values, enabling multicolumn index use without a filter on the leading column**

```sql
-- PostgreSQL 18: Skip scan for multicolumn indexes
CREATE INDEX idx_orders_status_date ON orders(status, created_at);

-- Skip scan iterates through distinct 'status' values internally,
-- performing a range lookup on created_at within each group.
-- Most effective when the leading column has few distinct values:
EXPLAIN SELECT * FROM orders WHERE created_at > '2024-01-01';
-- The planner may show: Index Scan using idx_orders_status_date
-- with a Skip Scan prefix for each distinct status value
```


## Resources

- [Index Types](https://www.postgresql.org/docs/18/indexes-types.html) — All PostgreSQL index types

---

> 📘 *This lesson is part of the [PostgreSQL Performance Engineering](https://stanza.dev/courses/postgresql-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*