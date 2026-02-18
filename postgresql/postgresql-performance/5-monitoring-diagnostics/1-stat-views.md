---
source_course: "postgresql-performance"
source_lesson: "postgresql-performance-pg-stat-views"
---

# PostgreSQL Statistics Views

PostgreSQL collects extensive statistics through pg_stat_* views.

## Activity Monitoring

### pg_stat_activity
Current connections and queries:

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

-- Find long-running queries (> 5 minutes)
SELECT *
FROM pg_stat_activity
WHERE state = 'active'
  AND NOW() - query_start > INTERVAL '5 minutes';

-- Kill a runaway query
SELECT pg_terminate_backend(pid);
```

## Table Statistics

### pg_stat_user_tables
```sql
-- Table activity overview
SELECT 
    relname AS table,
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

**Key metrics:**
- High `seq_scan` with low `idx_scan`: Missing indexes
- High `seq_tup_read`: Large sequential scans
- High `n_dead_tup`: Needs vacuum

### Tables needing indexes
```sql
-- Tables with many sequential scans reading many rows
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

## Index Statistics

### pg_stat_user_indexes
```sql
-- Index usage
SELECT 
    indexrelname AS index,
    relname AS table,
    idx_scan AS scans,
    idx_tup_read,
    idx_tup_fetch,
    pg_size_pretty(pg_relation_size(indexrelid)) AS size
FROM pg_stat_user_indexes
ORDER BY idx_scan DESC;

-- Unused indexes (candidates for removal)
SELECT 
    indexrelname AS index,
    relname AS table,
    pg_size_pretty(pg_relation_size(indexrelid)) AS size
FROM pg_stat_user_indexes
WHERE idx_scan = 0
  AND indexrelid NOT IN (
      SELECT conindid FROM pg_constraint WHERE contype IN ('p', 'u')
  )
ORDER BY pg_relation_size(indexrelid) DESC;
```

## Database Statistics

```sql
-- Overall database stats
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

**Target:** Cache hit ratio > 99%

📖 [Statistics Collector](https://www.postgresql.org/docs/18/monitoring-stats.html)

## Resources

- [Monitoring Statistics](https://www.postgresql.org/docs/18/monitoring-stats.html) — Statistics monitoring system

---

> 📘 *This lesson is part of the [PostgreSQL Performance Engineering](https://stanza.dev/courses/postgresql-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*