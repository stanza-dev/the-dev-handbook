---
source_course: "postgresql-performance"
source_lesson: "postgresql-performance-vacuum-operations"
---

# VACUUM Operations

VACUUM is essential for PostgreSQL health.

## Standard VACUUM

Reclaims space from dead tuples:

```sql
-- Basic vacuum
VACUUM orders;

-- Verbose output
VACUUM VERBOSE orders;

-- Vacuum all tables in database
VACUUM;
```

**What it does:**
1. Scans for dead tuples
2. Marks space as reusable
3. Updates visibility map
4. Updates statistics (with ANALYZE)

**What it doesn't do:**
- Return space to OS
- Shrink table files

## VACUUM FULL

Completely rewrites the table:

```sql
-- Warning: Takes exclusive lock!
VACUUM FULL orders;
```

**Use sparingly because:**
- Requires ACCESS EXCLUSIVE lock (blocks all queries)
- Needs disk space for a full copy
- Can take hours on large tables

## VACUUM ANALYZE

```sql
-- Vacuum and update statistics
VACUUM ANALYZE orders;
```

Equivalent to running VACUUM then ANALYZE separately.

## Autovacuum

PostgreSQL automatically vacuums tables:

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

### Autovacuum Triggers

Autovacuum runs when:

```
dead_tuples > vacuum_threshold + (scale_factor × table_size)
```

Default: `50 + (0.2 × rows)` = vacuum when 20% of table is dead

### Tuning Autovacuum

```sql
-- Per-table settings for busy tables
ALTER TABLE orders SET (
    autovacuum_vacuum_scale_factor = 0.05,  -- 5% instead of 20%
    autovacuum_analyze_scale_factor = 0.02,
    autovacuum_vacuum_cost_limit = 1000     -- More aggressive
);
```

## Transaction ID Wraparound

PostgreSQL uses 32-bit transaction IDs that wrap around:

```sql
-- Check age of oldest transaction ID
SELECT 
    datname,
    age(datfrozenxid) AS xid_age,
    current_setting('autovacuum_freeze_max_age')::bigint AS freeze_max
FROM pg_database
ORDER BY age(datfrozenxid) DESC;
```

**If XID age approaches 2 billion**, PostgreSQL will shut down to prevent data corruption!

VACUUM "freezes" old transactions to prevent this.

📖 [Routine Vacuuming](https://www.postgresql.org/docs/18/routine-vacuuming.html)

## Resources

- [VACUUM](https://www.postgresql.org/docs/18/sql-vacuum.html) — VACUUM command reference

---

> 📘 *This lesson is part of the [PostgreSQL Performance Engineering](https://stanza.dev/courses/postgresql-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*