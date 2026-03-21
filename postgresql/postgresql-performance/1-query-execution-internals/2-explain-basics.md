---
source_course: "postgresql-performance"
source_lesson: "postgresql-performance-explain-basics"
---

# EXPLAIN Fundamentals

## Introduction

EXPLAIN is your primary tool for understanding query performance. It reveals the execution plan the planner chose and, with ANALYZE, shows actual runtime metrics. Mastering EXPLAIN output is the single most important skill for PostgreSQL performance work.

## Key Concepts

- **EXPLAIN**: Shows the planned execution strategy without running the query.
- **EXPLAIN ANALYZE**: Actually executes the query and reports real timing and row counts alongside the estimates.
- **Plan node**: A single operation in the execution tree (e.g., Seq Scan, Hash Join).
- **Startup cost vs total cost**: Work before the first row vs work for all rows.
- **Loops**: How many times a node was executed (important for nested loops).

## Real World Context

When a production query takes 30 seconds instead of 30 milliseconds, EXPLAIN ANALYZE is how you find out why. It shows you exactly where time is spent - whether it is a missing index causing a sequential scan, a bad join strategy, or stale statistics producing wrong row estimates.

## Deep Dive

### Basic EXPLAIN

The simplest form shows the planned strategy without executing the query:

```sql
EXPLAIN SELECT * FROM users WHERE email = 'alice@example.com';
```

The output reveals which scan type was chosen and the estimated cost:

```
                           QUERY PLAN
-----------------------------------------------------------------
 Index Scan using users_email_idx on users
   (cost=0.42..8.44 rows=1 width=128)
   Index Cond: (email = 'alice@example.com'::text)
```

### EXPLAIN ANALYZE

Adding ANALYZE actually executes the query and shows real timing:

```sql
EXPLAIN ANALYZE 
SELECT * FROM orders o
JOIN users u ON o.user_id = u.id
WHERE o.created_at > '2024-01-01';
```

The output now includes both estimated and actual metrics:

```
                                    QUERY PLAN
--------------------------------------------------------------------------------
 Hash Join  (cost=15.25..127.89 rows=453 width=192)
            (actual time=0.521..2.143 rows=512 loops=1)
   Hash Cond: (o.user_id = u.id)
   ->  Seq Scan on orders o  (cost=0.00..103.75 rows=453 width=64)
                              (actual time=0.012..0.987 rows=512 loops=1)
         Filter: (created_at > '2024-01-01'::date)
         Rows Removed by Filter: 1488
   ->  Hash  (cost=10.50..10.50 rows=380 width=128)
              (actual time=0.489..0.490 rows=380 loops=1)
         Buckets: 1024  Batches: 1  Memory Usage: 57kB
         ->  Seq Scan on users u  (cost=0.00..10.50 rows=380 width=128)
                                   (actual time=0.008..0.223 rows=380 loops=1)
 Planning Time: 0.285 ms
 Execution Time: 2.231 ms
```

Comparing estimated rows (453) to actual rows (512) tells you whether statistics are accurate.

### Understanding Plan Structure

The plan is a tree of nodes, read from bottom to top and inside out:

```
Hash Join           ← Top node (final operation)
├── Seq Scan (orders)    ← Left child (outer relation)
└── Hash               ← Right child (inner relation)
    └── Seq Scan (users)
```

Child nodes execute first and feed their output to parent nodes.

### Key Metrics

| Metric | Description |
|--------|-------------|
| `cost=start..total` | Estimated cost in arbitrary units |
| `rows=N` | Estimated rows returned |
| `width=N` | Average row size in bytes |
| `actual time=start..total` | Real execution time (ms) |
| `rows=N` (actual) | Actual rows returned |
| `loops=N` | Times this node was executed |

### EXPLAIN Options

In PostgreSQL 18, EXPLAIN ANALYZE automatically includes buffer usage information (previously you needed to specify BUFFERS explicitly):

```sql
-- In PostgreSQL 18, EXPLAIN ANALYZE includes BUFFERS by default
EXPLAIN ANALYZE SELECT * FROM orders WHERE user_id = 42;
-- Equivalent to: EXPLAIN (ANALYZE, BUFFERS) in older versions

-- To disable buffer info:
EXPLAIN (ANALYZE, BUFFERS OFF) SELECT * FROM orders WHERE user_id = 42;
```

Other useful option combinations:

```sql
-- Show timing breakdown per operation
EXPLAIN (ANALYZE, TIMING) SELECT * FROM orders;

-- JSON format (for visualization tools like pgMustard, explain.depesz.com)
EXPLAIN (ANALYZE, FORMAT JSON) SELECT * FROM orders;
```

These options give you different views into query performance.

## Common Pitfalls

1. **Using EXPLAIN without ANALYZE for performance debugging** - Plain EXPLAIN shows estimates only. The actual execution time and row counts from ANALYZE are essential for diagnosing real problems.
2. **Ignoring the loops column** - A node with `actual time=0.1ms` but `loops=10000` actually took 1000ms total. Multiply time by loops for the true cost.
3. **Running EXPLAIN ANALYZE on destructive statements** - EXPLAIN ANALYZE executes the query. Wrap UPDATE/DELETE in a transaction and ROLLBACK if you just want to see the plan.

## Best Practices

1. **Always use EXPLAIN (ANALYZE, BUFFERS)** - Buffer information reveals whether queries hit cache or read from disk, which is critical for understanding real-world performance.
2. **Compare estimated vs actual rows** - Large discrepancies (10x or more) indicate stale or insufficient statistics. Run ANALYZE to fix them.
3. **Use visualization tools** - Paste EXPLAIN output into tools like explain.depesz.com or pgMustard for easier interpretation of complex plans.

## Summary

- EXPLAIN shows the planned execution strategy; EXPLAIN ANALYZE shows actual runtime metrics.
- In PostgreSQL 18, EXPLAIN ANALYZE includes buffer information by default.
- Plan nodes form a tree read from bottom to top.
- Compare estimated vs actual rows to detect statistics problems.
- Always wrap EXPLAIN ANALYZE of write queries in a transaction to avoid side effects.

## Code Examples

**Using EXPLAIN ANALYZE to see actual execution metrics, and wrapping destructive queries in a transaction for safety**

```sql
-- PostgreSQL 18: EXPLAIN ANALYZE includes BUFFERS by default
EXPLAIN ANALYZE
SELECT * FROM orders o
JOIN users u ON o.user_id = u.id
WHERE o.created_at > '2024-01-01';

-- To see the plan for a destructive query without executing:
BEGIN;
EXPLAIN ANALYZE DELETE FROM orders WHERE created_at < '2020-01-01';
ROLLBACK;
```


## Resources

- [Using EXPLAIN](https://www.postgresql.org/docs/18/using-explain.html) — Complete guide to EXPLAIN

---

> 📘 *This lesson is part of the [PostgreSQL Performance Engineering](https://stanza.dev/courses/postgresql-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*