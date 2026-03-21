---
source_course: "postgresql-performance"
source_lesson: "postgresql-performance-sub-partitioning"
---

# Sub-partitioning and Partition Strategies

## Introduction

Single-level partitioning covers most use cases, but some workloads require multi-level partitioning - dividing partitions into further sub-partitions. Choosing the right partition granularity and strategy is crucial: too many partitions waste planner time and memory, while too few negate the performance benefits.

## Key Concepts

- **Sub-partitioning (multi-level partitioning)**: Partitioning a partition into further child tables using a different key.
- **Partition granularity**: The size of each partition's range (e.g., daily, monthly, yearly).
- **Partition-wise join**: A planner feature that joins partitions pairwise for better parallelism.
- **Declarative partitioning**: PostgreSQL's built-in partitioning syntax (preferred over inheritance-based).

## Real World Context

A global e-commerce platform might partition orders by month (range) and then sub-partition each month by region (list). This enables efficient queries that filter by both time and region, and allows dropping old data per region independently. Choosing monthly vs daily granularity depends on data volume and query patterns.

## Deep Dive

### Multi-level Partitioning

You can partition a partition using a different key and method. Here is a range-by-date then list-by-region setup:

```sql
CREATE TABLE sales (
    id SERIAL,
    region TEXT,
    amount NUMERIC,
    sold_at TIMESTAMP
) PARTITION BY RANGE (sold_at);

-- First level: monthly range partitions
CREATE TABLE sales_2024_01 PARTITION OF sales
    FOR VALUES FROM ('2024-01-01') TO ('2024-02-01')
    PARTITION BY LIST (region);

-- Second level: sub-partitions by region
CREATE TABLE sales_2024_01_us PARTITION OF sales_2024_01
    FOR VALUES IN ('US');
CREATE TABLE sales_2024_01_eu PARTITION OF sales_2024_01
    FOR VALUES IN ('EU');
CREATE TABLE sales_2024_01_apac PARTITION OF sales_2024_01
    FOR VALUES IN ('APAC');
```

With this structure, a query filtering by both date range and region prunes to a single sub-partition.

### Choosing Partition Granularity

Granularity determines how many partitions you manage and how effective pruning is:

```sql
-- Too coarse: yearly partitions on high-volume table
-- Each partition is huge, limited pruning benefit
CREATE TABLE logs_2024 PARTITION OF logs
    FOR VALUES FROM ('2024-01-01') TO ('2025-01-01');

-- Too fine: daily partitions
-- 365 partitions per year, planner overhead, management burden
CREATE TABLE logs_2024_01_01 PARTITION OF logs
    FOR VALUES FROM ('2024-01-01') TO ('2024-01-02');

-- Just right for most cases: monthly partitions
-- 12 per year, good pruning, manageable count
CREATE TABLE logs_2024_01 PARTITION OF logs
    FOR VALUES FROM ('2024-01-01') TO ('2024-02-01');
```

The right granularity depends on data volume per period and typical query time ranges. A good rule of thumb is to keep individual partitions between 1GB and 100GB.

### Performance Considerations: Partition Count

PostgreSQL's planner evaluates all partitions during planning, so having too many partitions increases planning time:

```sql
-- Check planning time with many partitions
EXPLAIN ANALYZE SELECT * FROM logs WHERE created_at = '2024-06-15';
-- Planning Time: 15.2 ms  (with 1000+ daily partitions)
-- Planning Time: 0.8 ms   (with 12 monthly partitions)
```

Keep the total partition count under a few hundred for best planning performance. If you need more granularity, consider sub-partitioning: 12 monthly partitions, each with 4 hash sub-partitions gives 48 total - far better than 365 daily partitions.

### PG18 Partitioning Improvements

PostgreSQL 18 brings several improvements to partitioning performance:

```sql
-- PG18: Faster partition pruning with many partitions
-- The planner uses a more efficient algorithm for
-- identifying relevant partitions, reducing planning time
-- for tables with hundreds of partitions.

-- PG18: Improved join performance with partitioned tables
SET enable_partitionwise_join = on;
-- Partition-wise joins now handle more complex cases,
-- including joins where partition keys are not identical
-- but are compatible.
```

These improvements make partitioning more viable for complex schemas.

## Common Pitfalls

1. **Sub-partitioning without need** - Multi-level partitioning adds complexity. Only use it when queries consistently filter on two dimensions (e.g., time AND region).
2. **Creating daily partitions on low-volume tables** - If each partition holds only a few thousand rows, the overhead of partition management exceeds the pruning benefit.
3. **Mixing partitioning with heavy cross-partition queries** - Queries that aggregate across all partitions (e.g., total sales for all months and regions) may run slower than on a non-partitioned table due to Append node overhead.

## Best Practices

1. **Start with single-level monthly partitioning** - Add sub-partitioning only when you have a proven need for two-dimensional pruning.
2. **Keep total partition count under a few hundred** - Use fewer, larger partitions when possible to minimize planning overhead.
3. **Use partition-wise joins and aggregates** - Enable `enable_partitionwise_join` and `enable_partitionwise_aggregate` to let PostgreSQL process partitions in parallel.

## Summary

- Multi-level partitioning divides data along two dimensions (e.g., time then region).
- Monthly granularity is the best starting point for most time-series workloads.
- Too many partitions increase planning time; too few reduce pruning effectiveness.
- Keep total partition count under a few hundred for optimal planner performance.
- PostgreSQL 18 improves partition pruning efficiency and partition-wise join handling.

## Code Examples

**Setting up range-then-list sub-partitioning for two-dimensional partition pruning**

```sql
-- Multi-level partitioning: range by date, then list by region
CREATE TABLE sales (
    id SERIAL,
    region TEXT,
    amount NUMERIC,
    sold_at TIMESTAMP
) PARTITION BY RANGE (sold_at);

CREATE TABLE sales_2024_01 PARTITION OF sales
    FOR VALUES FROM ('2024-01-01') TO ('2024-02-01')
    PARTITION BY LIST (region);

CREATE TABLE sales_2024_01_us PARTITION OF sales_2024_01
    FOR VALUES IN ('US');
CREATE TABLE sales_2024_01_eu PARTITION OF sales_2024_01
    FOR VALUES IN ('EU');

-- Query prunes to single sub-partition:
SELECT * FROM sales
WHERE sold_at = '2024-01-15' AND region = 'US';
```


## Resources

- [Partitioning Best Practices](https://www.postgresql.org/docs/18/ddl-partitioning.html#DDL-PARTITIONING-DECLARATIVE) — Declarative partitioning strategies and best practices

---

> 📘 *This lesson is part of the [PostgreSQL Performance Engineering](https://stanza.dev/courses/postgresql-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*