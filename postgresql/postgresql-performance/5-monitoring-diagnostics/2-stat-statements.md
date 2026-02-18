---
source_course: "postgresql-performance"
source_lesson: "postgresql-performance-pg-stat-statements"
---

# pg_stat_statements

The essential extension for identifying slow queries.

## Installation

```sql
-- Enable the extension
CREATE EXTENSION pg_stat_statements;
```

```ini
# postgresql.conf
shared_preload_libraries = 'pg_stat_statements'
pg_stat_statements.max = 10000
pg_stat_statements.track = all
```

**Requires restart** to load the library.

## Finding Slow Queries

```sql
-- Top queries by total time
SELECT 
    ROUND(total_exec_time::numeric, 2) AS total_ms,
    calls,
    ROUND(mean_exec_time::numeric, 2) AS avg_ms,
    ROUND((100.0 * total_exec_time / 
           SUM(total_exec_time) OVER ())::numeric, 2) AS pct,
    LEFT(query, 80) AS query
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 20;
```

## Different Perspectives

### By Total Time (Most Impact)
```sql
SELECT query, total_exec_time, calls
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 10;
```

### By Average Time (Slowest Individual Queries)
```sql
SELECT query, mean_exec_time, calls
FROM pg_stat_statements
WHERE calls > 100  -- Exclude rarely run queries
ORDER BY mean_exec_time DESC
LIMIT 10;
```

### By Call Count (Most Frequent)
```sql
SELECT query, calls, total_exec_time
FROM pg_stat_statements
ORDER BY calls DESC
LIMIT 10;
```

## Query Normalization

pg_stat_statements normalizes queries:

```sql
-- These are tracked as ONE query:
SELECT * FROM users WHERE id = 1;
SELECT * FROM users WHERE id = 42;
SELECT * FROM users WHERE id = 999;

-- Shown as:
-- SELECT * FROM users WHERE id = $1
```

## Analyzing I/O

```sql
-- Queries with most buffer reads (I/O intensive)
SELECT 
    query,
    calls,
    shared_blks_read + shared_blks_hit AS total_buffers,
    shared_blks_read AS disk_reads,
    ROUND(100.0 * shared_blks_hit / 
          NULLIF(shared_blks_read + shared_blks_hit, 0), 2) AS hit_pct
FROM pg_stat_statements
ORDER BY shared_blks_read DESC
LIMIT 10;
```

## Resetting Statistics

```sql
-- Reset all statistics
SELECT pg_stat_statements_reset();

-- Reset for specific user/database
SELECT pg_stat_statements_reset(userid, dbid, queryid);
```

📖 [pg_stat_statements](https://www.postgresql.org/docs/18/pgstatstatements.html)

## Resources

- [pg_stat_statements](https://www.postgresql.org/docs/18/pgstatstatements.html) — Track execution statistics

---

> 📘 *This lesson is part of the [PostgreSQL Performance Engineering](https://stanza.dev/courses/postgresql-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*