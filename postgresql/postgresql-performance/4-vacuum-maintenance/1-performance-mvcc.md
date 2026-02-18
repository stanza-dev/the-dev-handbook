---
source_course: "postgresql-performance"
source_lesson: "postgresql-performance-mvcc"
---

# Multi-Version Concurrency Control (MVCC)

MVCC is PostgreSQL's core mechanism for handling concurrent access without locking.

## How MVCC Works

When you UPDATE or DELETE, PostgreSQL doesn't immediately remove the old data:

```
Row Version Timeline:

┌─────────────────────────────────────────────────────────────┐
│  Transaction 100: INSERT INTO users (id, name) VALUES (1, 'Alice')  │
│                                                             │
│  Row: id=1, name='Alice', xmin=100, xmax=0                 │
└─────────────────────────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│  Transaction 200: UPDATE users SET name='Bob' WHERE id=1   │
│                                                             │
│  Old Row: id=1, name='Alice', xmin=100, xmax=200 (dead)    │
│  New Row: id=1, name='Bob', xmin=200, xmax=0 (live)        │
└─────────────────────────────────────────────────────────────┘
```

**Key concepts:**
- `xmin`: Transaction ID that created this row version
- `xmax`: Transaction ID that deleted/updated this row version
- **Dead tuples**: Old row versions no longer visible to any transaction

## Why Dead Tuples Accumulate

1. Every UPDATE creates a new row version
2. Every DELETE marks the row as dead
3. Old versions are kept for concurrent transactions
4. Only VACUUM can reclaim the space

## Table Bloat

Without VACUUM, tables grow indefinitely:

```sql
-- Check for bloat
SELECT 
    relname AS table,
    n_dead_tup AS dead_tuples,
    n_live_tup AS live_tuples,
    ROUND(100.0 * n_dead_tup / NULLIF(n_live_tup + n_dead_tup, 0), 2) AS dead_pct
FROM pg_stat_user_tables
WHERE n_dead_tup > 0
ORDER BY n_dead_tup DESC;
```

## Impact of Bloat

1. **Larger table size**: More disk space
2. **Slower sequential scans**: Must read dead rows
3. **Inefficient indexes**: Point to dead tuples
4. **Worse cache efficiency**: Dead data in memory

## The Visibility Map

Tracks which pages contain only visible tuples:

- **All-visible**: Page has no dead tuples, can skip in VACUUM
- **All-frozen**: All tuples are old enough to be "frozen"

```sql
-- Check visibility map coverage
SELECT 
    relname,
    pg_size_pretty(pg_relation_size(oid)) AS size,
    COALESCE(ROUND(100.0 * pg_stat_get_live_tuples(oid) / 
             NULLIF(pg_class.reltuples, 0), 2), 0) AS live_pct
FROM pg_class
WHERE relkind = 'r'
ORDER BY pg_relation_size(oid) DESC
LIMIT 10;
```

📖 [MVCC Introduction](https://www.postgresql.org/docs/18/mvcc-intro.html)

## Resources

- [MVCC](https://www.postgresql.org/docs/18/mvcc.html) — Concurrency control documentation

---

> 📘 *This lesson is part of the [PostgreSQL Performance Engineering](https://stanza.dev/courses/postgresql-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*