---
source_course: "postgresql-performance"
source_lesson: "postgresql-performance-planner-config"
---

# Planner Configuration

Tune the planner's cost model for your hardware and workload.

## Cost Parameters

### I/O Costs
```sql
-- Sequential page read (baseline = 1.0)
SET seq_page_cost = 1.0;

-- Random page read (default 4.0)
-- Lower for SSDs, higher for spinning disk
SET random_page_cost = 1.1;  -- SSD
SET random_page_cost = 4.0;  -- HDD
```

**Impact:** Lower `random_page_cost` encourages more index scans

### CPU Costs
```sql
-- Cost per row processed
SET cpu_tuple_cost = 0.01;

-- Cost per index entry processed
SET cpu_index_tuple_cost = 0.005;

-- Cost per operator/function evaluation
SET cpu_operator_cost = 0.0025;
```

### Effective Cache Size
```sql
-- How much memory available for disk caching
-- Set to ~75% of total RAM for dedicated server
SET effective_cache_size = '24GB';
```

**Impact:** Higher values encourage index scans over sequential scans

## Enabling/Disabling Plan Types

For debugging (not production!):

```sql
-- Disable sequential scans (force index use)
SET enable_seqscan = off;

-- Disable specific join types
SET enable_hashjoin = off;
SET enable_mergejoin = off;
SET enable_nestloop = off;

-- Re-enable
SET enable_seqscan = on;
```

## Parallelism Settings

```sql
-- Maximum parallel workers per query
SET max_parallel_workers_per_gather = 4;

-- Minimum table size to consider parallel scan
SET min_parallel_table_scan_size = '8MB';

-- Minimum index size for parallel index scan
SET min_parallel_index_scan_size = '512kB';
```

## Memory Settings

```sql
-- Memory for sorts, hash joins (per operation)
SET work_mem = '256MB';

-- Memory for maintenance (VACUUM, CREATE INDEX)
SET maintenance_work_mem = '1GB';
```

**Caution with work_mem:**
- Applied per-sort-operation, not per-query
- A query with 10 sorts uses 10 × work_mem
- Too high can exhaust memory

## JIT Compilation

```sql
-- Enable JIT compilation
SET jit = on;

-- Costs to trigger JIT
SET jit_above_cost = 100000;
SET jit_inline_above_cost = 500000;
SET jit_optimize_above_cost = 500000;
```

JIT can significantly speed up complex analytical queries.

## Session vs Global Settings

```sql
-- Session only
SET work_mem = '512MB';

-- Permanent (in postgresql.conf or ALTER SYSTEM)
ALTER SYSTEM SET random_page_cost = 1.1;
SELECT pg_reload_conf();  -- Apply without restart
```

📖 [Query Planning](https://www.postgresql.org/docs/18/runtime-config-query.html)

## Resources

- [Query Planning Config](https://www.postgresql.org/docs/18/runtime-config-query.html) — Query planner configuration

---

> 📘 *This lesson is part of the [PostgreSQL Performance Engineering](https://stanza.dev/courses/postgresql-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*