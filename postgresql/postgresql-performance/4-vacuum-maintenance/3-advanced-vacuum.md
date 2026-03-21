---
source_course: "postgresql-performance"
source_lesson: "postgresql-performance-advanced-vacuum"
---

# Advanced VACUUM Strategies

## Introduction

Standard VACUUM handles most maintenance needs, but high-throughput tables require advanced strategies. `pg_repack` provides lock-free table compaction, HOT updates reduce dead tuple creation, and fine-tuned autovacuum monitoring ensures you catch problems before they impact users. PostgreSQL 18 brings further improvements with eager freezing controls and fixed dead tuple thresholds.

## Key Concepts

- **pg_repack**: A third-party extension that rewrites tables online without exclusive locks, serving as a practical alternative to VACUUM FULL.
- **HOT update (Heap-Only Tuple)**: An optimization where PostgreSQL updates a row in-place on the same page, avoiding index updates.
- **fillfactor**: The percentage of each table page to fill with data, leaving room for HOT updates.
- **Autovacuum workers**: Background processes that run VACUUM and ANALYZE on tables exceeding dead tuple thresholds.

## Real World Context

A 500GB orders table with 30% bloat needs compaction, but VACUUM FULL would lock it for hours. `pg_repack` compacts it online with minimal locking. Meanwhile, setting `fillfactor = 80` on a frequently-updated status column reduces dead tuples by 50% through HOT updates, cutting autovacuum frequency in half.

## Deep Dive

### pg_repack: Online Table Rewrites

`pg_repack` is the practical alternative to VACUUM FULL. It rewrites tables without holding an exclusive lock for the duration:

```sql
-- Install the extension
CREATE EXTENSION pg_repack;
```

Use it from the command line:

```sql
-- Repack a specific table (run from shell, not psql)
-- pg_repack -d mydb -t bloated_table

-- Repack all tables in a database
-- pg_repack -d mydb

-- Repack only indexes
-- pg_repack -d mydb -t orders --only-indexes
```

pg_repack works by creating a new copy of the table, replaying changes via a trigger, then swapping the tables atomically. The exclusive lock is held only during the final swap (milliseconds).

### HOT Updates and fillfactor

HOT (Heap-Only Tuple) updates are an optimization where PostgreSQL can update a row without creating new index entries, as long as the new row version fits on the same page and no indexed columns change:

```sql
-- Set fillfactor to leave room for HOT updates
ALTER TABLE orders SET (fillfactor = 80);

-- Now VACUUM to rewrite with the new fillfactor
VACUUM FULL orders;  -- or use pg_repack

-- Check HOT update effectiveness
SELECT 
    relname,
    n_tup_upd AS total_updates,
    n_tup_hot_upd AS hot_updates,
    ROUND(100.0 * n_tup_hot_upd / NULLIF(n_tup_upd, 0), 2) AS hot_pct
FROM pg_stat_user_tables
WHERE n_tup_upd > 0
ORDER BY n_tup_upd DESC;
```

A high HOT update percentage means fewer dead tuples and less index maintenance. Setting fillfactor to 70-80% is recommended for heavily updated tables.

### Monitoring Autovacuum Performance

Track whether autovacuum is keeping up with your workload:

```sql
-- Tables where autovacuum is falling behind
SELECT 
    relname,
    n_dead_tup,
    last_autovacuum,
    autovacuum_count,
    EXTRACT(EPOCH FROM (NOW() - last_autovacuum)) / 3600 AS hours_since_vacuum
FROM pg_stat_user_tables
WHERE n_dead_tup > 10000
ORDER BY n_dead_tup DESC;

-- Check if autovacuum workers are running
SELECT pid, datname, relid::regclass, phase, 
       heap_blks_total, heap_blks_scanned
FROM pg_stat_progress_vacuum;
```

If autovacuum consistently falls behind, increase `autovacuum_max_workers` or raise the cost limit per table.

### Tuning for High-Throughput Tables

For tables with millions of updates per day, the default autovacuum settings are too conservative:

```sql
-- Aggressive autovacuum for high-throughput table
ALTER TABLE high_write_table SET (
    autovacuum_vacuum_scale_factor = 0.01,   -- 1% dead tuples
    autovacuum_analyze_scale_factor = 0.005,
    autovacuum_vacuum_cost_delay = 2,        -- Less delay between I/O
    autovacuum_vacuum_cost_limit = 2000,     -- More I/O per round
    fillfactor = 80                          -- Room for HOT updates
);
```

These settings make autovacuum more aggressive on this specific table without affecting others.

### PG18 Vacuum Improvements

PostgreSQL 18 adds two important vacuum controls:

```sql
-- PG18: Eager freezing is enabled by default (0.03 = 3% failure tolerance)
-- Normal vacuums proactively freeze all-visible pages, reducing the
-- need for costly anti-wraparound vacuums later.
SHOW vacuum_max_eager_freeze_failure_rate;  -- Default: 0.03
-- Higher values = more persistent freezing attempts
-- 0.0 = DISABLES eager freezing entirely (not recommended)

-- PG18: Fixed dead tuple count threshold
ALTER TABLE huge_table SET (
    autovacuum_vacuum_max_threshold = 500000
);
-- On a billion-row table, 1% scale factor = 10M dead tuples.
-- max_threshold caps it at 500K regardless of table size.
```

The max_threshold setting is particularly useful for very large tables where even small percentages translate to millions of dead tuples.

## Common Pitfalls

1. **Using VACUUM FULL instead of pg_repack** - VACUUM FULL holds an exclusive lock for the entire duration, which can be hours for large tables. pg_repack achieves the same result with only milliseconds of exclusive locking.
2. **Not monitoring HOT update ratios** - If your hot_pct is low on frequently updated tables, you are creating unnecessary dead tuples. Check whether indexed columns are being updated or if fillfactor needs adjustment.
3. **Setting fillfactor too low** - A fillfactor of 50% wastes half your disk space. 70-80% is the sweet spot for most update-heavy workloads.

## Best Practices

1. **Use pg_repack instead of VACUUM FULL** - For production compaction, pg_repack provides the same space reclamation without extended locking.
2. **Set fillfactor = 80 on update-heavy tables** - Leave room for HOT updates to reduce dead tuple creation and index maintenance.
3. **Monitor autovacuum lag** - Check `pg_stat_progress_vacuum` and dead tuple counts. If autovacuum is falling behind, increase workers or raise cost limits per table.

## Summary

- pg_repack compacts tables online without the exclusive locking of VACUUM FULL.
- HOT updates avoid creating new index entries when the update fits on the same page and no indexed columns change.
- Setting fillfactor to 70-80% enables more HOT updates on frequently updated tables.
- Monitor autovacuum performance with dead tuple counts and vacuum progress views.
- PG18 eager freezing and `autovacuum_vacuum_max_threshold` provide finer control over vacuum behavior.

## Code Examples

**Using pg_repack for online compaction and monitoring HOT update effectiveness with fillfactor**

```sql
-- Use pg_repack instead of VACUUM FULL for online compaction
CREATE EXTENSION pg_repack;
-- Then from shell: pg_repack -d mydb -t bloated_table

-- Enable HOT updates with fillfactor
ALTER TABLE orders SET (fillfactor = 80);

-- Monitor HOT update effectiveness
SELECT relname,
       n_tup_hot_upd AS hot_updates,
       ROUND(100.0 * n_tup_hot_upd / NULLIF(n_tup_upd, 0), 2) AS hot_pct
FROM pg_stat_user_tables
WHERE n_tup_upd > 0 ORDER BY hot_pct;
```


## Resources

- [Routine Vacuuming](https://www.postgresql.org/docs/18/routine-vacuuming.html) — Comprehensive guide to routine vacuuming and autovacuum tuning

---

> 📘 *This lesson is part of the [PostgreSQL Performance Engineering](https://stanza.dev/courses/postgresql-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*