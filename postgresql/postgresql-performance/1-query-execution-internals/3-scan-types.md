---
source_course: "postgresql-performance"
source_lesson: "postgresql-performance-scan-types"
---

# Understanding Scan Types

The scan type determines how PostgreSQL reads data from tables.

## Sequential Scan (Seq Scan)

Reads **every row** in the table:

```sql
EXPLAIN SELECT * FROM large_table WHERE status = 'active';
```
```
Seq Scan on large_table  (cost=0.00..25000.00 rows=50000 width=100)
  Filter: (status = 'active')
```

**When it's used:**
- No suitable index exists
- Retrieving a large percentage of rows (>5-10%)
- Table is small enough to fit in memory

**When it's bad:**
- Large table, finding few rows
- Frequent queries for specific values

## Index Scan

Uses an index to find rows, then fetches from the table:

```sql
EXPLAIN SELECT * FROM users WHERE email = 'alice@example.com';
```
```
Index Scan using users_email_idx on users  (cost=0.42..8.44 rows=1 width=128)
  Index Cond: (email = 'alice@example.com')
```

**Characteristics:**
- Reads index → finds row locations → fetches rows from heap
- Efficient for selective queries (few rows)
- Returns rows in index order

## Index Only Scan

The **fastest scan type** - reads only from the index:

```sql
-- If index includes (user_id, email)
EXPLAIN SELECT user_id, email FROM users WHERE user_id = 42;
```
```
Index Only Scan using users_pkey on users  (cost=0.29..4.31 rows=1 width=36)
  Index Cond: (user_id = 42)
```

**Requirements:**
- All columns in SELECT/WHERE are in the index
- Table's visibility map is up-to-date (vacuum)

## Bitmap Index Scan

Combines index lookup with efficient heap access:

```sql
EXPLAIN SELECT * FROM orders 
WHERE status = 'pending' AND total > 100;
```
```
Bitmap Heap Scan on orders  (cost=12.89..387.32 rows=250 width=64)
  Recheck Cond: (status = 'pending')
  Filter: (total > 100)
  ->  Bitmap Index Scan on orders_status_idx  (cost=0.00..12.82 rows=500 width=0)
        Index Cond: (status = 'pending')
```

**How it works:**
1. Build bitmap of matching row locations
2. Sort locations by physical position
3. Fetch rows in sequential order (efficient I/O)

**When used:**
- Multiple conditions can be combined (BitmapAnd, BitmapOr)
- More rows than Index Scan handles efficiently
- Fewer rows than justify Seq Scan

## Scan Type Comparison

| Scan Type | Best For | I/O Pattern |
|-----------|----------|-------------|
| Seq Scan | Large result sets, small tables | Sequential |
| Index Scan | Few rows, ordered results | Random |
| Index Only Scan | All columns in index | Index only |
| Bitmap Scan | Medium result sets, multiple conditions | Sorted random |

📖 [Performance Tips](https://www.postgresql.org/docs/18/performance-tips.html)

---

> 📘 *This lesson is part of the [PostgreSQL Performance Engineering](https://stanza.dev/courses/postgresql-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*