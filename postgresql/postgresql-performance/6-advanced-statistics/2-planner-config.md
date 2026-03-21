---
source_course: "postgresql-performance"
source_lesson: "postgresql-performance-planner-config"
---

# Planner Configuration

## Introduction

PostgreSQL's planner uses a cost model that can be tuned to match your hardware and workload. Incorrect default settings - especially `random_page_cost` on SSDs - cause the planner to consistently choose suboptimal plans. Understanding and tuning these settings is essential for getting the best performance from your hardware.

## Key Concepts

- **random_page_cost**: The estimated cost of a random disk page read relative to sequential reads. Default 4.0 is for HDDs; SSDs should use 1.1-1.5.
- **effective_cache_size**: The total memory available for disk caching (OS cache + shared_buffers). Helps the planner decide between index and sequential scans.
- **work_mem**: Memory allocated per sort/hash operation. Affects whether sorts spill to disk.
- **Parallelism settings**: Controls for how many workers can be used for parallel scans, joins, and aggregates.
- **JIT compilation**: Just-in-time compilation of query expressions for complex analytical queries.

## Real World Context

A common production mistake: running PostgreSQL on SSDs with default `random_page_cost = 4.0`. This tells the planner that random reads are 4x more expensive than sequential reads - true for spinning disks, but not SSDs where random and sequential reads are nearly equal. The result: the planner avoids index scans even when they would be faster.

## Deep Dive

### I/O Cost Parameters

These settings control how the planner estimates I/O costs:

```sql
-- Sequential page read (baseline = 1.0)
SET seq_page_cost = 1.0;

-- Random page read (default 4.0)
-- Lower for SSDs, keep high for spinning disk
SET random_page_cost = 1.1;  -- SSD
SET random_page_cost = 4.0;  -- HDD
```

Lowering `random_page_cost` on SSDs encourages more index scans, which is the correct behavior when random I/O is nearly as fast as sequential.

### CPU Cost Parameters

Fine-tune CPU cost estimates:

```sql
SET cpu_tuple_cost = 0.01;         -- Cost per row processed
SET cpu_index_tuple_cost = 0.005;  -- Cost per index entry processed
SET cpu_operator_cost = 0.0025;    -- Cost per operator/function evaluation
```

These rarely need adjustment, but can matter for CPU-bound analytical queries.

### Effective Cache Size

Tell the planner how much memory is available for caching:

```sql
-- Set to ~75% of total RAM for dedicated server
SET effective_cache_size = '24GB';
```

Higher values make the planner more likely to choose index scans because it assumes more data will be in cache.

### Enabling/Disabling Plan Types

For debugging only (not production), you can force or prevent specific plan types:

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

These `enable_*` flags are invaluable for debugging: if disabling sequential scans makes your query faster, you know an index is needed.

### Parallelism Settings

Control parallel query execution:

```sql
-- Maximum parallel workers per query
SET max_parallel_workers_per_gather = 4;

-- Minimum table size to consider parallel scan
SET min_parallel_table_scan_size = '8MB';

-- Minimum index size for parallel index scan
SET min_parallel_index_scan_size = '512kB';
```

Parallel queries can dramatically speed up scans on large tables.

### Memory Settings

Memory affects whether operations stay in RAM or spill to disk:

```sql
-- Memory for sorts, hash joins (per operation)
SET work_mem = '256MB';

-- Memory for maintenance (VACUUM, CREATE INDEX)
SET maintenance_work_mem = '1GB';
```

Caution: work_mem is allocated per sort or hash operation, not per query. A query with 10 sorts uses 10 x work_mem.

### JIT Compilation

JIT can speed up complex analytical queries:

```sql
SET jit = on;
SET jit_above_cost = 100000;
SET jit_inline_above_cost = 500000;
SET jit_optimize_above_cost = 500000;
```

JIT compiles query expressions to native code, most beneficial for queries processing millions of rows with complex expressions.

### Session vs Global Settings

Settings can be applied at different scopes:

```sql
-- Session only
SET work_mem = '512MB';

-- Permanent (in postgresql.conf or ALTER SYSTEM)
ALTER SYSTEM SET random_page_cost = 1.1;
SELECT pg_reload_conf();  -- Apply without restart
```

Use session-level settings for testing and ALTER SYSTEM for permanent changes.

## Common Pitfalls

1. **Default random_page_cost on SSDs** - The default of 4.0 is calibrated for spinning disks. On SSDs, this discourages index scans that would be beneficial.
2. **Setting work_mem too high globally** - A setting of 1GB with 100 concurrent connections running complex queries could attempt to allocate 100GB+ of memory.
3. **Using enable_* flags in production** - These are debugging tools. Disabling seqscan or hashjoin in production can cause catastrophic performance for queries that genuinely need those strategies.

## Best Practices

1. **Set random_page_cost = 1.1 for SSDs** - This is the single most impactful tuning change for SSD-based PostgreSQL deployments.
2. **Set work_mem per query, not globally** - Use `SET LOCAL work_mem = '256MB'` in transactions that need it, rather than raising the global default.
3. **Set effective_cache_size to 75% of total RAM** - This gives the planner an accurate picture of caching capability.

## Summary

- random_page_cost should be 1.1-1.5 for SSDs (default 4.0 is for HDDs).
- effective_cache_size should reflect total available cache (OS + shared_buffers).
- work_mem is per-operation, not per-query - set it carefully.
- enable_* flags are debugging tools, not production settings.
- JIT compilation benefits complex analytical queries processing millions of rows.

## Code Examples

**Key planner configuration changes for SSD storage, per-transaction work_mem, and debugging with enable_* flags**

```sql
-- Essential SSD configuration
ALTER SYSTEM SET random_page_cost = 1.1;
ALTER SYSTEM SET effective_cache_size = '24GB';  -- 75% of RAM
SELECT pg_reload_conf();

-- Per-transaction work_mem for a large sort
BEGIN;
SET LOCAL work_mem = '256MB';
SELECT * FROM huge_table ORDER BY complex_expression;
COMMIT;

-- Debug: force index scan to test performance
SET enable_seqscan = off;
EXPLAIN ANALYZE SELECT * FROM orders WHERE user_id = 42;
SET enable_seqscan = on;  -- Re-enable immediately!
```


## Resources

- [Query Planning Config](https://www.postgresql.org/docs/18/runtime-config-query.html) — Query planner configuration

---

> 📘 *This lesson is part of the [PostgreSQL Performance Engineering](https://stanza.dev/courses/postgresql-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*