---
source_course: "postgresql-performance"
source_lesson: "postgresql-performance-query-optimization"
---

# Query Optimization Patterns

## Introduction

Beyond indexes and statistics, there are query-level optimization patterns that can dramatically improve performance. PostgreSQL 18 introduces automatic self-join elimination, but understanding join order optimization, recognizing anti-patterns, and knowing how to debug plan choices are skills that remain essential across all versions.

## Key Concepts

- **Self-join elimination**: PG18's ability to automatically remove redundant joins where a table is joined to itself unnecessarily.
- **Join order optimization**: The planner's strategy for determining the most efficient order to join multiple tables.
- **N+1 query problem**: An anti-pattern where an application executes one query per row from a parent query, leading to thousands of round trips.
- **Implicit cast**: An automatic type conversion that can prevent index usage.
- **enable_* flags**: Planner settings for debugging that force or prevent specific plan types.

## Real World Context

ORM-generated queries often contain redundant self-joins, implicit type casts, and N+1 patterns that developers do not notice until production load exposes them. PostgreSQL 18's self-join elimination fixes some of these automatically, but understanding the full range of anti-patterns lets you catch problems before they reach production.

## Deep Dive

### PG18: Self-Join Elimination

PostgreSQL 18 can automatically detect and remove redundant self-joins. This commonly occurs with ORM-generated queries or views that reference the same table multiple times:

```sql
-- Before PG18: This query joins orders to itself unnecessarily
SELECT o1.id, o1.total, o2.status
FROM orders o1
JOIN orders o2 ON o1.id = o2.id
WHERE o1.user_id = 42;

-- PG18 automatically eliminates the self-join:
-- Internally optimized to:
-- SELECT id, total, status FROM orders WHERE user_id = 42;
```

The planner recognizes that `o1` and `o2` reference the same rows (joined on the primary key) and collapses them into a single table access. This is especially beneficial for complex views that compose multiple CTEs or subqueries referencing the same table.

### Join Order Optimization

With many tables, the number of possible join orders grows factorially. PostgreSQL uses different strategies based on table count:

```sql
-- With few tables (< 12), PostgreSQL evaluates all possible orders
-- With many tables, it uses the Genetic Query Optimizer (GEQO)
SHOW geqo_threshold;  -- Default: 12

-- For complex queries, you can hint at join order:
-- (not real hints - use join_collapse_limit instead)
SET join_collapse_limit = 1;  -- Preserve explicit JOIN order
```

When `join_collapse_limit = 1`, PostgreSQL respects the order you wrote your JOINs in. This is useful when you know the optimal join order but the planner chooses poorly.

### Common Query Anti-Patterns

**N+1 Queries**: The most common ORM performance problem:

```sql
-- Anti-pattern: one query per user's orders
-- Application code loops through users, executing for each:
SELECT * FROM orders WHERE user_id = 1;
SELECT * FROM orders WHERE user_id = 2;
-- ... repeated 1000 times

-- Fix: single query with JOIN or IN
SELECT u.id, u.name, o.total
FROM users u
JOIN orders o ON o.user_id = u.id
WHERE u.active = true;
```

One query with a JOIN replaces thousands of individual queries.

**Implicit Casts preventing index usage**:

```sql
-- Anti-pattern: implicit cast prevents index usage
-- Column user_id is INTEGER, but parameter is TEXT
SELECT * FROM orders WHERE user_id = '42';
-- PostgreSQL may cast every row's user_id to text for comparison

-- Fix: use the correct type
SELECT * FROM orders WHERE user_id = 42;
```

Always ensure parameter types match column types to avoid implicit casts that prevent index usage.

**Unnecessary DISTINCT or ORDER BY**:

```sql
-- Anti-pattern: DISTINCT on a column that's already unique
SELECT DISTINCT id, name FROM users WHERE active = true;
-- 'id' is a primary key - DISTINCT is redundant but adds sort cost

-- Fix: remove unnecessary DISTINCT
SELECT id, name FROM users WHERE active = true;
```

Unnecessary sorts waste CPU and memory.

### Using enable_* Flags for Debugging

When a query uses a surprising plan, use `enable_*` flags to test alternatives:

```sql
-- Step 1: See the current plan
EXPLAIN ANALYZE SELECT * FROM orders WHERE status = 'pending';

-- Step 2: Force index usage to compare
SET enable_seqscan = off;
EXPLAIN ANALYZE SELECT * FROM orders WHERE status = 'pending';
SET enable_seqscan = on;

-- Step 3: If index plan is faster, investigate why planner chose seqscan
-- Common causes: stale statistics, wrong random_page_cost, low selectivity
```

This process helps you understand why the planner made its choice and what needs to change.

### PG18: HAVING Clause Pushdown for GROUPING SETS

PostgreSQL 18 optimizes queries with GROUPING SETS by pushing HAVING conditions down:

```sql
-- PG18: HAVING pushdown for GROUPING SETS
SELECT region, product, SUM(sales) AS total
FROM orders
GROUP BY GROUPING SETS ((region), (product), ())
HAVING SUM(sales) > 10000;

-- In PG18, the HAVING filter is applied during grouping
-- rather than after, reducing intermediate result size
```

This optimization reduces memory usage and execution time for complex aggregation queries.

## Common Pitfalls

1. **N+1 queries from ORMs** - Always check the SQL your ORM generates. Enable query logging temporarily to count queries per request. A single page load should not execute hundreds of queries.
2. **Type mismatches in WHERE clauses** - Comparing a varchar column with an integer parameter forces PostgreSQL to cast every row, preventing index usage.
3. **Over-using DISTINCT** - DISTINCT adds a sort or hash aggregate. Only use it when duplicates are actually possible and undesirable.

## Best Practices

1. **Use EXPLAIN ANALYZE on ORM-generated queries** - Do not trust the ORM to generate efficient SQL. Verify with EXPLAIN ANALYZE.
2. **Match parameter types to column types** - Use explicit casts when necessary: `WHERE user_id = $1::integer`.
3. **Debug with enable_* flags, then fix the root cause** - Use flags to understand the problem, then fix statistics, indexes, or cost parameters rather than leaving flags set.

## Summary

- PostgreSQL 18 automatically eliminates redundant self-joins, especially from ORM-generated queries.
- Join order matters: use `join_collapse_limit` when you know the optimal order.
- N+1 queries, implicit casts, and unnecessary DISTINCT/ORDER BY are the most common anti-patterns.
- Use `enable_*` flags to test alternative plans, then fix the root cause.
- PG18 HAVING pushdown for GROUPING SETS reduces intermediate result sizes.

## Code Examples

**PG18 self-join elimination, fixing implicit cast anti-patterns, and debugging plan choices with enable_* flags**

```sql
-- PG18: Self-join elimination
-- Before: redundant join (common in ORM-generated queries)
SELECT o1.id, o1.total, o2.status
FROM orders o1 JOIN orders o2 ON o1.id = o2.id
WHERE o1.user_id = 42;
-- PG18 optimizes to: SELECT id, total, status FROM orders WHERE user_id = 42

-- Anti-pattern: implicit cast prevents index usage
SELECT * FROM orders WHERE user_id = '42';  -- TEXT vs INTEGER
-- Fix: match types
SELECT * FROM orders WHERE user_id = 42;

-- Debug plan choices with enable_* flags
SET enable_seqscan = off;
EXPLAIN ANALYZE SELECT * FROM orders WHERE status = 'pending';
SET enable_seqscan = on;
```


## Resources

- [Query Performance Tips](https://www.postgresql.org/docs/18/performance-tips.html) — PostgreSQL performance optimization tips and techniques

---

> 📘 *This lesson is part of the [PostgreSQL Performance Engineering](https://stanza.dev/courses/postgresql-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*