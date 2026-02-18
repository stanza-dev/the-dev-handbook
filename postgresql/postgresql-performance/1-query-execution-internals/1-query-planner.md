---
source_course: "postgresql-performance"
source_lesson: "postgresql-performance-query-planner"
---

# The PostgreSQL Query Planner

Before executing any query, PostgreSQL's query planner creates an **execution plan** - a step-by-step strategy for retrieving data efficiently.

## The Query Processing Pipeline

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

## What the Planner Does

The planner evaluates **multiple strategies** for executing your query:

1. **Scan methods**: How to read from tables (sequential, index, bitmap)
2. **Join methods**: How to combine tables (nested loop, hash, merge)
3. **Join order**: Which tables to join first
4. **Aggregation strategies**: How to group and aggregate

## Cost-Based Optimization

PostgreSQL uses **cost-based optimization**. It estimates:

- **Disk I/O**: Reading pages from disk
- **CPU**: Processing rows and evaluating conditions
- **Memory**: Sorting, hashing operations

Costs are expressed in arbitrary units (roughly equal to a sequential page read).

```sql
-- See the planner's cost estimate
EXPLAIN SELECT * FROM orders WHERE user_id = 42;
```

Output:
```
                         QUERY PLAN
------------------------------------------------------------
 Index Scan using idx_orders_user on orders
   (cost=0.29..8.31 rows=1 width=64)
   Index Cond: (user_id = 42)
```

**Cost breakdown**: `cost=0.29..8.31`
- `0.29`: Startup cost (before first row)
- `8.31`: Total cost (for all rows)

## Statistics: The Foundation of Good Plans

The planner relies on **table statistics** to estimate:
- Number of rows
- Data distribution
- Column correlations

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

**Stale statistics = Bad plans!** Autovacuum updates statistics automatically, but after bulk operations, run ANALYZE manually.

📖 [Query Planning Overview](https://www.postgresql.org/docs/18/planner-optimizer.html)

## Resources

- [Query Planning](https://www.postgresql.org/docs/18/planner-optimizer.html) — How PostgreSQL plans queries

---

> 📘 *This lesson is part of the [PostgreSQL Performance Engineering](https://stanza.dev/courses/postgresql-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*