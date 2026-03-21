---
source_course: "postgresql-performance"
source_lesson: "postgresql-performance-pg-stat-statements"
---

# pg_stat_statements

## Introduction

pg_stat_statements is the essential extension for identifying slow queries in PostgreSQL. It tracks execution statistics for every distinct query, including total time, call count, mean time, and buffer I/O. Without it, finding your slowest queries requires parsing log files - with it, a single SQL query gives you the answer.

## Key Concepts

- **pg_stat_statements**: A contrib extension that records execution statistics for all SQL statements.
- **Query normalization**: The process of replacing literal values with parameters ($1, $2) so that identical query patterns are tracked as one entry.
- **Total time**: The cumulative execution time across all calls - the key metric for overall impact.
- **Mean time**: Average execution time per call - useful for finding individually slow queries.
- **Buffer hit ratio**: The percentage of blocks served from shared_buffers vs read from disk.

## Real World Context

In production, a query running in 10ms but called 100,000 times per hour has more total impact than a query running in 5 seconds but called once per day. pg_stat_statements reveals both patterns, letting you prioritize optimization work by total impact rather than individual query speed.

## Deep Dive

### Installation

Enable the extension in your database:

```sql
CREATE EXTENSION pg_stat_statements;
```

And configure it in postgresql.conf:

```
shared_preload_libraries = 'pg_stat_statements'
pg_stat_statements.max = 10000
pg_stat_statements.track = all
```

A server restart is required to load the library.

### Finding Slow Queries

The most useful query sorts by total execution time:

```sql
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

This shows the top 20 queries by total time, with their percentage of overall database time.

### Different Analysis Perspectives

Total time shows overall impact:

```sql
SELECT query, total_exec_time, calls
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 10;
```

Mean time reveals individually slow queries:

```sql
SELECT query, mean_exec_time, calls
FROM pg_stat_statements
WHERE calls > 100  -- Exclude rarely run queries
ORDER BY mean_exec_time DESC
LIMIT 10;
```

Call count identifies the most frequently executed queries:

```sql
SELECT query, calls, total_exec_time
FROM pg_stat_statements
ORDER BY calls DESC
LIMIT 10;
```

Each perspective reveals different optimization opportunities.

### Query Normalization

pg_stat_statements normalizes queries by replacing literal values with parameters:

```sql
-- These are tracked as ONE normalized query:
SELECT * FROM users WHERE id = 1;
SELECT * FROM users WHERE id = 42;
SELECT * FROM users WHERE id = 999;

-- Normalized form:
-- SELECT * FROM users WHERE id = $1
```

This aggregation is what makes pg_stat_statements useful - without it, you would have millions of individual query entries.

### Analyzing I/O Patterns

Find queries with the most disk reads:

```sql
SELECT 
    LEFT(query, 80) AS query,
    calls,
    shared_blks_read + shared_blks_hit AS total_buffers,
    shared_blks_read AS disk_reads,
    ROUND(100.0 * shared_blks_hit / 
          NULLIF(shared_blks_read + shared_blks_hit, 0), 2) AS hit_pct
FROM pg_stat_statements
ORDER BY shared_blks_read DESC
LIMIT 10;
```

Queries with low hit_pct are reading from disk and would benefit from better indexing or more shared_buffers.

### Resetting Statistics

Reset after making optimizations to measure improvement:

```sql
-- Reset all statistics
SELECT pg_stat_statements_reset();
```

Reset periodically (e.g., weekly) to keep the data relevant to current workload patterns.

## Common Pitfalls

1. **Not installing pg_stat_statements** - This extension is not enabled by default. It should be the first thing you install on any PostgreSQL production database.
2. **Optimizing by mean time instead of total time** - A slow query running once daily matters less than a moderately slow query running millions of times.
3. **Forgetting to reset after optimization** - Old statistics can mask improvements. Reset after making changes to get accurate before/after comparisons.

## Best Practices

1. **Sort by total_exec_time for prioritization** - Always start with the queries consuming the most total time.
2. **Filter by calls > 100 for mean time analysis** - Exclude infrequent queries to focus on patterns that affect users.
3. **Export statistics before resetting** - Save a snapshot to a table or file before calling pg_stat_statements_reset() so you do not lose historical data.

## Summary

- pg_stat_statements tracks execution statistics for every normalized query.
- Sort by total_exec_time to find queries with the most overall impact.
- Query normalization groups identical patterns with different parameter values.
- Buffer hit ratio per query reveals I/O-bound queries that need better indexing.
- Reset statistics periodically and after optimizations to keep data relevant.

## Code Examples

**Finding the highest-impact queries by total execution time using pg_stat_statements**

```sql
-- Top queries by total execution time (overall impact)
SELECT 
    ROUND(total_exec_time::numeric, 2) AS total_ms,
    calls,
    ROUND(mean_exec_time::numeric, 2) AS avg_ms,
    ROUND((100.0 * total_exec_time / 
           SUM(total_exec_time) OVER ())::numeric, 2) AS pct_of_total,
    LEFT(query, 80) AS query
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 10;
```


## Resources

- [pg_stat_statements](https://www.postgresql.org/docs/18/pgstatstatements.html) — Track execution statistics

---

> 📘 *This lesson is part of the [PostgreSQL Performance Engineering](https://stanza.dev/courses/postgresql-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*