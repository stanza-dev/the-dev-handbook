---
source_course: "postgresql-performance"
source_lesson: "postgresql-performance-vacuum-operations"
---

# VACUUM Operations

## Introduction

VACUUM is essential for PostgreSQL health. It reclaims space from dead tuples, updates the visibility map, refreshes statistics, and prevents transaction ID wraparound. Understanding the different VACUUM variants and how to configure autovacuum is critical for maintaining production database performance.

## Key Concepts

- **VACUUM**: Marks dead tuple space as reusable without returning it to the OS.
- **VACUUM FULL**: Completely rewrites the table, reclaiming all space but requiring an exclusive lock.
- **VACUUM ANALYZE**: Combines vacuuming with statistics refresh.
- **Autovacuum**: A background process that automatically vacuums tables when dead tuple thresholds are exceeded.
- **Transaction ID wraparound**: A dangerous condition where PostgreSQL runs out of transaction IDs, potentially requiring a shutdown.

## Real World Context

In a production system processing thousands of updates per second, autovacuum runs continuously in the background to prevent bloat. Misconfigured autovacuum is one of the top causes of PostgreSQL performance degradation. Understanding the formula that triggers autovacuum and how to tune it per table prevents outages.

## Deep Dive

### Standard VACUUM

Standard VACUUM reclaims space from dead tuples without locking the table for reads:

```sql
-- Basic vacuum
VACUUM orders;

-- Verbose output shows what was cleaned
VACUUM VERBOSE orders;

-- Vacuum all tables in database
VACUUM;
```

Standard VACUUM marks dead tuple space as reusable and updates the visibility map, but it does not return space to the operating system or shrink table files.

### VACUUM FULL

VACUUM FULL rewrites the entire table, compacting it to minimal size:

```sql
-- Warning: Takes exclusive lock!
VACUUM FULL orders;
```

Use VACUUM FULL sparingly because it requires an ACCESS EXCLUSIVE lock (blocking all reads and writes), needs enough disk space for a full copy of the table, and can take hours on large tables.

### VACUUM ANALYZE

Combines vacuuming with a statistics refresh:

```sql
VACUUM ANALYZE orders;
```

This is equivalent to running VACUUM then ANALYZE separately, and is the most common form used in maintenance scripts.

### Autovacuum

PostgreSQL automatically vacuums tables when dead tuple counts exceed a configurable threshold:

```sql
-- Check autovacuum status
SELECT 
    relname,
    last_vacuum,
    last_autovacuum,
    vacuum_count,
    autovacuum_count
FROM pg_stat_user_tables
ORDER BY last_autovacuum NULLS FIRST;
```

The autovacuum trigger formula is:

```
dead_tuples > vacuum_threshold + (scale_factor x table_size)
```

Default values: `50 + (0.2 x rows)` means VACUUM triggers when 20% of the table is dead.

### Tuning Autovacuum

For busy tables, the default 20% threshold may be too high. You can tune per table:

```sql
ALTER TABLE orders SET (
    autovacuum_vacuum_scale_factor = 0.05,  -- 5% instead of 20%
    autovacuum_analyze_scale_factor = 0.02,
    autovacuum_vacuum_cost_limit = 1000     -- More aggressive
);
```

Lowering the scale factor triggers autovacuum sooner, preventing bloat from building up.

### PostgreSQL 18: Eager Freezing and Max Threshold

PostgreSQL 18 introduces eager freezing, where normal vacuums can proactively freeze all-visible pages. This is controlled by `vacuum_max_eager_freeze_failure_rate`, which defaults to 0.03 (3%):

```sql
-- PG18: Eager freezing is ON by default (rate = 0.03 = 3%)
-- The rate controls how many all-visible-but-not-frozen pages can
-- fail to be frozen before vacuum stops trying eagerly.
-- Default 0.03 means vacuum tolerates up to 3% failure rate.
SHOW vacuum_max_eager_freeze_failure_rate;  -- Default: 0.03

-- Setting to 0.0 DISABLES eager freezing entirely:
-- SET vacuum_max_eager_freeze_failure_rate = 0.0;

-- Higher values make vacuum more persistent in freezing attempts:
SET vacuum_max_eager_freeze_failure_rate = 0.05;  -- 5% tolerance
```

With the default setting, normal vacuums proactively freeze pages, reducing the need for costly anti-wraparound vacuums later.

PG18 also adds a fixed dead tuple threshold for autovacuum, independent of table size:

```sql
-- PG18: Fixed dead tuple threshold for autovacuum
ALTER TABLE busy_table SET (
    autovacuum_vacuum_max_threshold = 100000  -- vacuum after 100k dead tuples
);
```

This is useful for very large tables where the percentage-based threshold (e.g., 5% of 1 billion rows = 50 million dead tuples) allows too much bloat.

### Transaction ID Wraparound

PostgreSQL uses 32-bit transaction IDs that can wrap around. Monitor the age of the oldest transaction ID:

```sql
SELECT 
    datname,
    age(datfrozenxid) AS xid_age,
    current_setting('autovacuum_freeze_max_age')::bigint AS freeze_max
FROM pg_database
ORDER BY age(datfrozenxid) DESC;
```

If XID age approaches 2 billion, PostgreSQL will shut down to prevent data corruption. VACUUM freezes old transactions to prevent this.

## Common Pitfalls

1. **Using VACUUM FULL as routine maintenance** - VACUUM FULL blocks all access and is almost never needed. Standard VACUUM or `pg_repack` (see next lesson) is preferable.
2. **Not monitoring autovacuum lag** - If autovacuum cannot keep up with your write rate, bloat accumulates silently. Check `last_autovacuum` timestamps regularly.
3. **Ignoring XID wraparound warnings** - Transaction ID wraparound is a real production risk. Set up monitoring for `age(datfrozenxid)` and alert well before it approaches `autovacuum_freeze_max_age`.

## Best Practices

1. **Tune autovacuum per table** - Busy tables need lower `autovacuum_vacuum_scale_factor` (0.01-0.05) and higher `autovacuum_vacuum_cost_limit`.
2. **Use PG18 max_threshold for large tables** - Set `autovacuum_vacuum_max_threshold` to cap the number of dead tuples regardless of table size.
3. **Monitor XID age** - Set up alerts when any database's `age(datfrozenxid)` exceeds 500 million.

## Summary

- Standard VACUUM marks dead space as reusable without locking the table.
- VACUUM FULL rewrites the table but requires an exclusive lock - use it sparingly.
- Autovacuum triggers based on dead tuple count relative to table size.
- PostgreSQL 18 adds eager freezing and `autovacuum_vacuum_max_threshold` for better control.
- Monitor XID age to prevent transaction ID wraparound shutdowns.

## Code Examples

**Tuning autovacuum per table, using PG18 features, and monitoring transaction ID wraparound**

```sql
-- Tune autovacuum for a high-write table
ALTER TABLE orders SET (
    autovacuum_vacuum_scale_factor = 0.05,
    autovacuum_vacuum_cost_limit = 1000
);

-- PG18: Fixed dead tuple threshold (for very large tables)
ALTER TABLE events SET (
    autovacuum_vacuum_max_threshold = 100000
);

-- PG18: Eager freezing is ON by default (0.03 = 3% failure tolerance)
-- Check current setting:
SHOW vacuum_max_eager_freeze_failure_rate;
-- Note: setting to 0.0 DISABLES eager freezing

-- Monitor XID wraparound risk
SELECT datname, age(datfrozenxid) AS xid_age
FROM pg_database ORDER BY xid_age DESC;
```


## Resources

- [VACUUM](https://www.postgresql.org/docs/18/sql-vacuum.html) — VACUUM command reference

---

> 📘 *This lesson is part of the [PostgreSQL Performance Engineering](https://stanza.dev/courses/postgresql-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*