---
source_course: "postgresql-performance"
source_lesson: "postgresql-performance-scan-types"
---

# Understanding Scan Types

## Introduction

The scan type determines how PostgreSQL reads data from tables. Choosing the right scan strategy is one of the most impactful decisions the planner makes. Understanding when and why each scan type is selected helps you design better indexes and write faster queries.

## Key Concepts

- **Sequential Scan (Seq Scan)**: Reads every row in the table from start to finish.
- **Index Scan**: Uses an index to find specific rows, then fetches them from the table heap.
- **Index Only Scan**: Reads all needed data directly from the index without visiting the table.
- **Bitmap Index Scan**: Builds a bitmap of matching row locations, then fetches them in physical order.

## Real World Context

A missing index on a frequently-queried column can cause PostgreSQL to sequentially scan millions of rows when it only needs a handful. Conversely, an unnecessary index scan on a large result set can be slower than a simple sequential scan. Knowing which scan type to expect lets you spot misconfigurations immediately.

## Deep Dive

### Sequential Scan (Seq Scan)

Reads every row in the table:

```sql
EXPLAIN SELECT * FROM large_table WHERE status = 'active';
```

The output shows a full table scan:

```
Seq Scan on large_table  (cost=0.00..25000.00 rows=50000 width=100)
  Filter: (status = 'active')
```

Sequential scans are used when no suitable index exists, when retrieving a large percentage of rows (typically above 5-10%), or when the table is small enough that an index adds overhead without benefit.

### Index Scan

Uses an index to locate rows, then fetches the full row from the heap:

```sql
EXPLAIN SELECT * FROM users WHERE email = 'alice@example.com';
```

This shows an efficient lookup through the index:

```
Index Scan using users_email_idx on users  (cost=0.42..8.44 rows=1 width=128)
  Index Cond: (email = 'alice@example.com')
```

Index scans are efficient for highly selective queries that return few rows and when results need to be returned in index order.

### Index Only Scan

The fastest scan type - reads everything from the index without touching the table heap:

```sql
-- If index includes (user_id, email)
EXPLAIN SELECT user_id, email FROM users WHERE user_id = 42;
```

The output confirms the heap is not accessed:

```
Index Only Scan using users_pkey on users  (cost=0.29..4.31 rows=1 width=36)
  Index Cond: (user_id = 42)
```

This requires that all columns in SELECT and WHERE are in the index, and that the table's visibility map is up-to-date (maintained by VACUUM).

### Bitmap Index Scan

Combines index lookup with efficient heap access by sorting row locations before fetching:

```sql
EXPLAIN SELECT * FROM orders 
WHERE status = 'pending' AND total > 100;
```

The two-phase approach is visible in the plan:

```
Bitmap Heap Scan on orders  (cost=12.89..387.32 rows=250 width=64)
  Recheck Cond: (status = 'pending')
  Filter: (total > 100)
  ->  Bitmap Index Scan on orders_status_idx  (cost=0.00..12.82 rows=500 width=0)
        Index Cond: (status = 'pending')
```

Bitmap scans work by first building a bitmap of matching row locations, then sorting those locations by physical position, and finally fetching rows in sequential order for efficient I/O. Multiple conditions can be combined using BitmapAnd and BitmapOr nodes.

### Scan Type Comparison

| Scan Type | Best For | I/O Pattern |
|-----------|----------|-------------|
| Seq Scan | Large result sets, small tables | Sequential |
| Index Scan | Few rows, ordered results | Random |
| Index Only Scan | All columns in index | Index only |
| Bitmap Scan | Medium result sets, multiple conditions | Sorted random |

## Common Pitfalls

1. **Assuming sequential scans are always bad** - For queries returning more than 5-10% of a table, a sequential scan is often faster than thousands of random index lookups.
2. **Expecting Index Only Scans without vacuuming** - Index Only Scans need an up-to-date visibility map. If VACUUM has not run recently, PostgreSQL falls back to a regular Index Scan.
3. **Creating too many indexes hoping for bitmap scans** - While bitmap scans can combine multiple indexes, each index has maintenance overhead. A single well-designed multi-column index often outperforms combining several single-column indexes.

## Best Practices

1. **Design indexes for your query patterns** - Create indexes that match your WHERE clauses and include frequently selected columns via INCLUDE for Index Only Scans.
2. **Monitor scan types in production** - Use `pg_stat_user_tables` to track the ratio of sequential scans to index scans per table.
3. **Let the planner choose** - Avoid forcing scan types with `enable_seqscan = off` in production. Trust the planner's cost model and fix the underlying statistics or indexes instead.

## Summary

- Sequential scans read entire tables and are appropriate for large result sets or small tables.
- Index scans use an index for selective lookups but incur random I/O.
- Index Only Scans are the fastest, reading data entirely from the index.
- Bitmap scans bridge the gap between index and sequential scans for medium selectivity.
- The planner automatically chooses the best scan type based on cost estimates and statistics.

## Code Examples

**Demonstrating how different query patterns lead to different scan types based on selectivity and available indexes**

```sql
-- Compare scan types with EXPLAIN
-- Sequential scan (large result set, no index)
EXPLAIN SELECT * FROM logs WHERE level = 'INFO';

-- Index scan (highly selective)
EXPLAIN SELECT * FROM users WHERE email = 'alice@example.com';

-- Index only scan (all columns in index)
EXPLAIN SELECT user_id FROM orders WHERE user_id = 42;

-- Bitmap scan (medium selectivity, combining conditions)
EXPLAIN SELECT * FROM orders 
WHERE status = 'pending' AND total > 100;
```


## Resources

- [Performance Tips](https://www.postgresql.org/docs/18/performance-tips.html) — PostgreSQL performance tips including scan type selection

---

> 📘 *This lesson is part of the [PostgreSQL Performance Engineering](https://stanza.dev/courses/postgresql-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*