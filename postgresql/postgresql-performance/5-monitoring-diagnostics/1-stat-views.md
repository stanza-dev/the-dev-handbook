---
source_course: "postgresql-performance"
source_lesson: "postgresql-performance-pg-stat-views"
---

# PostgreSQL Statistics Views

## Introduction

PostgreSQL collects extensive runtime statistics through the Cumulative Statistics System (renamed from "Statistics Collector" in PostgreSQL 15). These pg_stat_* views provide real-time insight into table activity, connection state, index usage, and I/O patterns. They are your primary tools for monitoring performance and identifying bottlenecks.

## Key Concepts

- **Cumulative Statistics System**: PostgreSQL's built-in system for collecting and reporting runtime statistics (formerly called the Statistics Collector).
- **pg_stat_activity**: Shows current connections, their state, and active queries.
- **pg_stat_user_tables**: Reports table-level metrics including scan counts, tuple operations, and dead tuple counts.
- **pg_stat_user_indexes**: Tracks index usage, helping identify unused indexes.
- **Cache hit ratio**: The percentage of data page requests served from shared_buffers rather than disk.

## Real World Context

When a production database slows down, the first place to look is `pg_stat_activity` for long-running queries and `pg_stat_user_tables` for tables with high sequential scan counts. Finding a table with 10 million sequential scans and zero index scans is a clear signal that an index is needed.

## Deep Dive

### Activity Monitoring with pg_stat_activity

This view shows all current connections and their state:

```sql
-- Active queries
SELECT 
    pid,
    usename,
    state,
    query_start,
    NOW() - query_start AS duration,
    LEFT(query, 80) AS query
FROM pg_stat_activity
WHERE state = 'active'
  AND pid <> pg_backend_pid()
ORDER BY query_start;
```

This query excludes your own session and shows active queries sorted by start time.

To find long-running queries that might be causing problems:

```sql
-- Find long-running queries (> 5 minutes)
SELECT pid, usename, query_start, LEFT(query, 100) AS query
FROM pg_stat_activity
WHERE state = 'active'
  AND NOW() - query_start > INTERVAL '5 minutes';

-- Kill a runaway query
SELECT pg_terminate_backend(pid);
```

Terminating a backend should be a last resort; prefer `pg_cancel_backend(pid)` which cancels the query without closing the connection.

### Table Statistics with pg_stat_user_tables

This view reveals how tables are being accessed:

```sql
SELECT 
    relname AS table_name,
    seq_scan,
    seq_tup_read,
    idx_scan,
    idx_tup_fetch,
    n_tup_ins AS inserts,
    n_tup_upd AS updates,
    n_tup_del AS deletes,
    n_dead_tup AS dead_tuples
FROM pg_stat_user_tables
ORDER BY seq_tup_read DESC
LIMIT 10;
```

Key metrics to watch: high `seq_scan` with low `idx_scan` indicates missing indexes, high `seq_tup_read` means large sequential scans, and high `n_dead_tup` means the table needs vacuuming.

To find tables that need indexes:

```sql
SELECT 
    relname,
    seq_scan,
    seq_tup_read,
    seq_tup_read / NULLIF(seq_scan, 0) AS avg_rows_per_scan,
    idx_scan
FROM pg_stat_user_tables
WHERE seq_scan > 100
  AND seq_tup_read / NULLIF(seq_scan, 0) > 10000
ORDER BY seq_tup_read DESC;
```

Tables with many sequential scans reading thousands of rows per scan are prime candidates for index creation.

### Index Statistics with pg_stat_user_indexes

Track which indexes are being used and which are waste:

```sql
-- Unused indexes (candidates for removal)
SELECT 
    indexrelname AS index_name,
    relname AS table_name,
    pg_size_pretty(pg_relation_size(indexrelid)) AS size
FROM pg_stat_user_indexes
WHERE idx_scan = 0
  AND indexrelid NOT IN (
      SELECT conindid FROM pg_constraint WHERE contype IN ('p', 'u')
  )
ORDER BY pg_relation_size(indexrelid) DESC;
```

This excludes primary key and unique constraint indexes, which are needed even if never explicitly scanned.

### Database-level Statistics

Monitor cache efficiency and overall activity:

```sql
SELECT 
    datname,
    numbackends AS connections,
    xact_commit AS commits,
    xact_rollback AS rollbacks,
    blks_read,
    blks_hit,
    ROUND(100.0 * blks_hit / NULLIF(blks_read + blks_hit, 0), 2) AS cache_hit_ratio
FROM pg_stat_database
WHERE datname = current_database();
```

Target a cache hit ratio above 99% for production databases.

### PG18: pg_stat_io Enhancements

PostgreSQL 18 enhances the `pg_stat_io` view with byte-level I/O tracking:

```sql
-- PG18: Detailed I/O statistics with byte counts
SELECT 
    backend_type,
    io_object,
    io_context,
    reads, read_bytes,
    writes, write_bytes,
    extends, extend_bytes
FROM pg_stat_io
WHERE reads > 0 OR writes > 0
ORDER BY read_bytes DESC;
```

The `read_bytes`, `write_bytes`, and `extend_bytes` columns provide more accurate I/O measurement than page counts alone.

## Common Pitfalls

1. **Only checking pg_stat_activity** - Connection state is just one dimension. Table and index statistics reveal structural problems like missing indexes.
2. **Dropping indexes with zero scans too quickly** - Indexes may be used only for periodic batch jobs or reporting queries. Check over a representative time period (at least a full business cycle) before dropping.
3. **Ignoring the cache hit ratio** - A drop in cache hit ratio often precedes visible performance problems. Monitor it continuously.

## Best Practices

1. **Set up continuous monitoring** - Export pg_stat_* metrics to a time-series database (Prometheus, InfluxDB) for trend analysis.
2. **Reset statistics after schema changes** - Use `pg_stat_reset()` after major index changes to get clean usage data.
3. **Check cache hit ratio daily** - A healthy production database should maintain above 99%. Below 95% indicates shared_buffers may be too small.

## Summary

- The Cumulative Statistics System (renamed from Statistics Collector in PG15) provides runtime performance data.
- `pg_stat_activity` shows active connections and queries; `pg_stat_user_tables` reveals table access patterns.
- High sequential scan counts with low index scan counts indicate missing indexes.
- Unused indexes waste disk space and slow down writes - remove them after verification.
- PG18 enhances `pg_stat_io` with byte-level I/O tracking for more accurate performance analysis.

## Code Examples

**Finding tables missing indexes and identifying unused indexes using PostgreSQL statistics views**

```sql
-- Find tables that likely need indexes
SELECT relname, seq_scan, idx_scan,
       seq_tup_read / NULLIF(seq_scan, 0) AS avg_rows_per_scan
FROM pg_stat_user_tables
WHERE seq_scan > 100
  AND seq_tup_read / NULLIF(seq_scan, 0) > 10000
ORDER BY seq_tup_read DESC;

-- Find unused indexes (candidates for removal)
SELECT indexrelname, relname,
       pg_size_pretty(pg_relation_size(indexrelid)) AS size
FROM pg_stat_user_indexes
WHERE idx_scan = 0
  AND indexrelid NOT IN (
    SELECT conindid FROM pg_constraint WHERE contype IN ('p','u')
  )
ORDER BY pg_relation_size(indexrelid) DESC;
```


## Resources

- [Monitoring Statistics](https://www.postgresql.org/docs/18/monitoring-stats.html) — Cumulative Statistics System documentation

---

> 📘 *This lesson is part of the [PostgreSQL Performance Engineering](https://stanza.dev/courses/postgresql-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*