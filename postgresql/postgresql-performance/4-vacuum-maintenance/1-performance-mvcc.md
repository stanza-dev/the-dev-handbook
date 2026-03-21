---
source_course: "postgresql-performance"
source_lesson: "postgresql-performance-mvcc"
---

# Multi-Version Concurrency Control (MVCC)

## Introduction

MVCC is PostgreSQL's core mechanism for handling concurrent access without readers blocking writers or vice versa. Instead of locking rows during reads, PostgreSQL keeps multiple versions of each row, allowing transactions to see consistent snapshots of the data. The trade-off is that old row versions accumulate and must be cleaned up by VACUUM.

## Key Concepts

- **MVCC (Multi-Version Concurrency Control)**: A concurrency model where each transaction sees a snapshot of the database, not the live state.
- **Dead tuple**: An old row version that is no longer visible to any active transaction.
- **Live tuple**: The current version of a row visible to new transactions.
- **Table bloat**: Wasted space from accumulated dead tuples that VACUUM has not yet reclaimed.
- **Visibility map**: A bitmap tracking which table pages contain only visible tuples.

## Real World Context

Every UPDATE in PostgreSQL creates a new row version and leaves the old one behind as a dead tuple. In a high-update workload (e.g., an e-commerce order status tracker), dead tuples can accumulate rapidly, doubling or tripling effective table size. Without VACUUM, sequential scans slow down, indexes bloat, and eventually you risk transaction ID wraparound.

## Deep Dive

### How MVCC Works

When you UPDATE or DELETE, PostgreSQL does not immediately remove the old data. Instead, it creates a new version:

```
Row Version Timeline:

┌─────────────────────────────────────────────────────────────┐
│  Transaction 100: INSERT INTO users (id, name) VALUES (1, 'Alice')  │
│  Row: id=1, name='Alice', xmin=100, xmax=0                 │
└─────────────────────────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│  Transaction 200: UPDATE users SET name='Bob' WHERE id=1   │
│  Old Row: id=1, name='Alice', xmin=100, xmax=200 (dead)    │
│  New Row: id=1, name='Bob', xmin=200, xmax=0 (live)        │
└─────────────────────────────────────────────────────────────┘
```

The `xmin` field records which transaction created this row version, and `xmax` records which transaction invalidated it. A row with `xmax=0` is still live.

### Why Dead Tuples Accumulate

Three factors drive dead tuple accumulation:

1. Every UPDATE creates a new row version and marks the old one dead
2. Every DELETE marks the row as dead
3. Old versions must be kept as long as any transaction might need them

Only VACUUM can reclaim the space occupied by dead tuples.

### Table Bloat

Without regular VACUUM, tables grow indefinitely. You can monitor bloat with this query:

```sql
SELECT 
    relname AS table,
    n_dead_tup AS dead_tuples,
    n_live_tup AS live_tuples,
    ROUND(100.0 * n_dead_tup / NULLIF(n_live_tup + n_dead_tup, 0), 2) AS dead_pct
FROM pg_stat_user_tables
WHERE n_dead_tup > 0
ORDER BY n_dead_tup DESC;
```

Tables with more than 10-20% dead tuples need attention.

### Impact of Bloat

Bloat causes four cascading problems:

1. **Larger table size**: More disk space consumed
2. **Slower sequential scans**: Must read dead rows alongside live ones
3. **Inefficient indexes**: Index entries point to dead tuples, wasting lookups
4. **Worse cache efficiency**: Dead data occupies shared_buffers, evicting useful data

### The Visibility Map

The visibility map tracks which pages contain only visible tuples:

```sql
SELECT 
    relname,
    pg_size_pretty(pg_relation_size(oid)) AS size
FROM pg_class
WHERE relkind = 'r'
ORDER BY pg_relation_size(oid) DESC
LIMIT 10;
```

Pages marked as all-visible can be skipped during VACUUM and enable Index Only Scans. Pages marked as all-frozen have all tuples old enough to be permanently visible.

## Common Pitfalls

1. **Assuming PostgreSQL handles bloat automatically** - Autovacuum helps, but under heavy write loads it may fall behind. Monitor dead tuple counts and tune autovacuum per table.
2. **Long-running transactions preventing cleanup** - A single idle transaction can prevent VACUUM from cleaning up dead tuples visible to that transaction's snapshot.
3. **Ignoring index bloat** - Indexes also accumulate dead entries. Standard VACUUM cleans the table but may not fully compact indexes; REINDEX or `pg_repack` may be needed.

## Best Practices

1. **Monitor dead tuple percentages** - Set up alerts when any table exceeds 20% dead tuples.
2. **Kill idle-in-transaction connections** - Set `idle_in_transaction_session_timeout` to prevent abandoned transactions from blocking VACUUM.
3. **Understand your update patterns** - Tables with frequent updates on the same rows (e.g., status fields) need more aggressive autovacuum settings.

## Summary

- MVCC provides concurrent access without locking by maintaining multiple row versions.
- UPDATE and DELETE create dead tuples that consume space until VACUUM reclaims them.
- Table bloat slows scans, wastes memory, and degrades index performance.
- The visibility map enables efficient VACUUM and Index Only Scans.
- Long-running transactions are the most common cause of VACUUM being unable to clean up dead tuples.

## Code Examples

**Monitoring dead tuple accumulation to identify tables that need VACUUM attention**

```sql
-- Check dead tuple accumulation across all tables
SELECT 
    relname AS table_name,
    n_dead_tup AS dead_tuples,
    n_live_tup AS live_tuples,
    ROUND(100.0 * n_dead_tup / NULLIF(n_live_tup + n_dead_tup, 0), 2) AS dead_pct,
    last_autovacuum
FROM pg_stat_user_tables
WHERE n_dead_tup > 1000
ORDER BY n_dead_tup DESC;
```


## Resources

- [MVCC](https://www.postgresql.org/docs/18/mvcc.html) — Concurrency control documentation

---

> 📘 *This lesson is part of the [PostgreSQL Performance Engineering](https://stanza.dev/courses/postgresql-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*