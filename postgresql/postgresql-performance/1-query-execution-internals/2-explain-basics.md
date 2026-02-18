---
source_course: "postgresql-performance"
source_lesson: "postgresql-performance-explain-basics"
---

# EXPLAIN Fundamentals

EXPLAIN is your primary tool for understanding query performance.

## Basic EXPLAIN

```sql
-- Show the execution plan (without running)
EXPLAIN SELECT * FROM users WHERE email = 'alice@example.com';
```

Output:
```
                           QUERY PLAN
-----------------------------------------------------------------
 Index Scan using users_email_idx on users
   (cost=0.42..8.44 rows=1 width=128)
   Index Cond: (email = 'alice@example.com'::text)
```

## EXPLAIN ANALYZE

**Actually executes the query** and shows real timing:

```sql
EXPLAIN ANALYZE 
SELECT * FROM orders o
JOIN users u ON o.user_id = u.id
WHERE o.created_at > '2024-01-01';
```

Output:
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

## Understanding the Output

### Plan Structure
The plan is a **tree of nodes**, read bottom-up:

```
Hash Join           ← Top node (final operation)
├── Seq Scan (orders)    ← Left child (outer relation)
└── Hash               ← Right child (inner relation)
    └── Seq Scan (users)
```

### Key Metrics

| Metric | Description |
|--------|-------------|
| `cost=start..total` | Estimated cost in arbitrary units |
| `rows=N` | Estimated rows returned |
| `width=N` | Average row size in bytes |
| `actual time=start..total` | Real execution time (ms) |
| `rows=N` | Actual rows returned |
| `loops=N` | Times this node was executed |

### Comparing Estimated vs Actual

```
Seq Scan on orders (cost=0.00..103.75 rows=453 width=64)
                   (actual time=0.012..0.987 rows=512 loops=1)
```

- **Estimated rows**: 453
- **Actual rows**: 512
- Close enough! Large differences indicate stale statistics.

## EXPLAIN Options

```sql
-- Include buffer usage information
EXPLAIN (ANALYZE, BUFFERS) SELECT ...;

-- Show timing breakdown per operation
EXPLAIN (ANALYZE, TIMING) SELECT ...;

-- All useful options combined
EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT) SELECT ...;

-- JSON format (for visualization tools)
EXPLAIN (ANALYZE, FORMAT JSON) SELECT ...;
```

📖 [Using EXPLAIN](https://www.postgresql.org/docs/18/using-explain.html)

## Resources

- [Using EXPLAIN](https://www.postgresql.org/docs/18/using-explain.html) — Complete guide to EXPLAIN

---

> 📘 *This lesson is part of the [PostgreSQL Performance Engineering](https://stanza.dev/courses/postgresql-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*