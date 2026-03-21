---
source_course: "postgresql-performance"
source_lesson: "postgresql-performance-query-planner"
---

# The PostgreSQL Query Planner

## Introduction

Before executing any query, PostgreSQL's query planner creates an execution plan - a step-by-step strategy for retrieving data efficiently. Understanding how the planner works is essential for writing performant queries and diagnosing slow ones.

## Key Concepts

- **Execution plan**: A tree of operations the database engine follows to retrieve the requested data.
- **Cost-based optimization**: The planner estimates the cost of multiple strategies and picks the cheapest one.
- **Table statistics**: Metadata about row counts, data distribution, and column correlations that guide the planner's estimates.
- **Startup cost**: Work required before the first result row can be returned.
- **Total cost**: Estimated work to return all result rows.

## Real World Context

Every SELECT, UPDATE, and DELETE you run passes through the planner. When a query suddenly becomes slow in production, it is almost always because the planner chose a poor execution strategy - often due to stale statistics or missing indexes. Understanding the planner lets you diagnose and fix these issues in minutes instead of hours.

## Deep Dive

### The Query Processing Pipeline

PostgreSQL processes every SQL statement through a multi-stage pipeline before any data is returned.

```
┌──────────────┐
│  SQL Query   │
└──────┬───────┘
       ▼
┌──────────────┐
│    Parser    │  → Syntax check, create parse tree
└──────┬───────┘
       ▼
┌──────────────┐
│   Analyzer   │  → Semantic analysis, create query tree
└──────┬───────┘
       ▼
┌──────────────┐
│   Rewriter   │  → Apply rules (views, etc.)
└──────┬───────┘
       ▼
┌──────────────┐
│   Planner    │  → Generate execution plan
└──────┬───────┘
       ▼
┌──────────────┐
│   Executor   │  → Run the plan, return results
└──────────────┘
```

The planner stage is where performance decisions are made. It evaluates multiple strategies for executing your query.

### What the Planner Evaluates

The planner considers four key dimensions when building a plan:

1. **Scan methods**: How to read from tables (sequential, index, bitmap)
2. **Join methods**: How to combine tables (nested loop, hash, merge)
3. **Join order**: Which tables to join first
4. **Aggregation strategies**: How to group and aggregate

### Cost-Based Optimization

PostgreSQL uses cost-based optimization to pick the best plan. It estimates three categories of cost:

- **Disk I/O**: Reading pages from disk
- **CPU**: Processing rows and evaluating conditions
- **Memory**: Sorting, hashing operations

Costs are expressed in arbitrary units roughly equal to one sequential page read. Here is how to see the planner's cost estimate:

```sql
EXPLAIN SELECT * FROM orders WHERE user_id = 42;
```

The output shows the chosen plan and its estimated cost:

```
                         QUERY PLAN
------------------------------------------------------------
 Index Scan using idx_orders_user on orders
   (cost=0.29..8.31 rows=1 width=64)
   Index Cond: (user_id = 42)
```

The cost breakdown `cost=0.29..8.31` tells you two things: `0.29` is the startup cost before the first row can be returned, and `8.31` is the total cost for all rows.

### Statistics: The Foundation of Good Plans

The planner relies on table statistics to estimate row counts, data distribution, and column correlations. Without accurate statistics, the planner may choose a catastrophically bad plan.

```sql
-- Update statistics for a table
ANALYZE orders;

-- View statistics
SELECT 
    relname,
    reltuples,     -- Estimated row count
    relpages       -- Number of disk pages
FROM pg_class 
WHERE relname = 'orders';
```

Stale statistics lead to bad plans. Autovacuum updates statistics automatically, but after bulk operations you should run ANALYZE manually.

## Common Pitfalls

1. **Ignoring stale statistics** - After large bulk loads or deletes, the planner may use outdated row count estimates. Always run `ANALYZE` after major data changes.
2. **Assuming the planner is wrong** - When a query uses a sequential scan instead of an index scan, it is often because the planner correctly determined that scanning the whole table is cheaper for the given data distribution.
3. **Not understanding cost units** - Cost values are not milliseconds. They are relative estimates useful only for comparing plans, not for predicting execution time.

## Best Practices

1. **Keep statistics fresh** - Ensure autovacuum is properly configured and runs frequently enough for your workload. Run `ANALYZE` manually after bulk operations.
2. **Read EXPLAIN output before optimizing** - Always check the execution plan before adding indexes or rewriting queries. The bottleneck may not be where you think.
3. **Use EXPLAIN (ANALYZE, BUFFERS) for real measurements** - Plain EXPLAIN shows estimates; EXPLAIN ANALYZE shows actual execution metrics.

## Summary

- The query planner evaluates multiple execution strategies and picks the cheapest one based on cost estimates.
- Costs are computed from table statistics, I/O estimates, and CPU estimates.
- Stale statistics are the most common cause of poor plan choices.
- EXPLAIN reveals the planner's chosen strategy and cost estimates.
- Autovacuum maintains statistics automatically, but manual ANALYZE is needed after bulk operations.

## Code Examples

**Using EXPLAIN to view the planner's chosen strategy and cost estimates for a filtered query**

```sql
-- See the planner's cost estimate for a simple query
EXPLAIN SELECT * FROM orders WHERE user_id = 42;

-- Output:
-- Index Scan using idx_orders_user on orders
--   (cost=0.29..8.31 rows=1 width=64)
--   Index Cond: (user_id = 42)

-- cost=0.29  → startup cost (work before first row)
-- cost=8.31  → total cost (work for all rows)
-- rows=1     → estimated number of rows returned
```

**Refreshing and inspecting table statistics that the planner uses for cost estimation**

```sql
-- Update statistics after a bulk load
ANALYZE orders;

-- Check table-level statistics
SELECT relname, reltuples, relpages
FROM pg_class
WHERE relname = 'orders';
-- reltuples: estimated row count
-- relpages:  number of 8kB disk pages
```


## Resources

- [Query Planning](https://www.postgresql.org/docs/18/planner-optimizer.html) — How PostgreSQL plans queries

---

> 📘 *This lesson is part of the [PostgreSQL Performance Engineering](https://stanza.dev/courses/postgresql-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*