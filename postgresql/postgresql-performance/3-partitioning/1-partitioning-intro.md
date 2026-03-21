---
source_course: "postgresql-performance"
source_lesson: "postgresql-performance-partitioning-intro"
---

# Table Partitioning

## Introduction

Partitioning divides one logical table into multiple physical tables for better performance and easier maintenance. When tables grow to hundreds of gigabytes or more, partitioning becomes essential for maintaining query performance, enabling efficient data lifecycle management, and reducing maintenance window duration.

## Key Concepts

- **Partitioned table**: A logical table that is physically stored as multiple child tables (partitions).
- **Partition key**: The column(s) used to determine which partition a row belongs to.
- **Partition pruning**: The planner's ability to skip partitions that cannot contain matching rows.
- **Range partitioning**: Divides data by value ranges (dates, numbers).
- **List partitioning**: Divides data by specific discrete values.
- **Hash partitioning**: Distributes data evenly by hash of the partition key.

## Real World Context

Time-series data is the most common use case: application logs, IoT sensor readings, financial transactions. Without partitioning, a table with a year of logs requires scanning the entire dataset even when you only need today's data. With monthly range partitioning, the planner touches only the relevant partition.

## Deep Dive

### Why Partition?

Partitioning provides two categories of benefits:

**Performance:**
- **Partition pruning**: Only scan relevant partitions
- **Parallel operations**: Process partitions simultaneously
- **Smaller indexes**: Each partition has smaller, more efficient indexes

**Maintenance:**
- **Drop old data instantly**: Detach and drop partition instead of slow DELETE
- **Vacuum individual partitions**: Less lock time per operation
- **Archive separately**: Move old partitions to slower storage

### Range Partitioning

Range partitioning divides data by value ranges, most commonly dates:

```sql
CREATE TABLE orders (
    id SERIAL,
    user_id INTEGER,
    total NUMERIC,
    created_at TIMESTAMP
) PARTITION BY RANGE (created_at);

CREATE TABLE orders_2024_01 PARTITION OF orders
    FOR VALUES FROM ('2024-01-01') TO ('2024-02-01');

CREATE TABLE orders_2024_02 PARTITION OF orders
    FOR VALUES FROM ('2024-02-01') TO ('2024-03-01');
```

The FROM bound is inclusive and the TO bound is exclusive, so there is no overlap between partitions.

### List Partitioning

List partitioning divides data by specific discrete values:

```sql
CREATE TABLE customers (
    id SERIAL,
    name TEXT,
    country TEXT
) PARTITION BY LIST (country);

CREATE TABLE customers_usa PARTITION OF customers
    FOR VALUES IN ('USA', 'US');

CREATE TABLE customers_europe PARTITION OF customers
    FOR VALUES IN ('UK', 'DE', 'FR', 'IT');

CREATE TABLE customers_asia PARTITION OF customers
    FOR VALUES IN ('JP', 'CN', 'KR');
```

List partitioning works well when data has natural categories like regions or tenants.

### Hash Partitioning

Hash partitioning distributes rows evenly using a hash function:

```sql
CREATE TABLE events (
    id SERIAL,
    user_id INTEGER,
    type TEXT,
    data JSONB
) PARTITION BY HASH (user_id);

CREATE TABLE events_0 PARTITION OF events
    FOR VALUES WITH (MODULUS 4, REMAINDER 0);
CREATE TABLE events_1 PARTITION OF events
    FOR VALUES WITH (MODULUS 4, REMAINDER 1);
CREATE TABLE events_2 PARTITION OF events
    FOR VALUES WITH (MODULUS 4, REMAINDER 2);
CREATE TABLE events_3 PARTITION OF events
    FOR VALUES WITH (MODULUS 4, REMAINDER 3);
```

Hash partitioning ensures even data distribution but does not support range queries on the partition key.

### Partition Pruning

Partition pruning is the key performance benefit. The planner skips partitions that cannot contain matching rows:

```sql
EXPLAIN SELECT * FROM orders WHERE created_at = '2024-01-15';
```

The plan shows only the relevant partition is scanned:

```
Append  (cost=0.00..25.88 rows=6 width=44)
  ->  Seq Scan on orders_2024_01  (cost=0.00..25.88 rows=6 width=44)
        Filter: (created_at = '2024-01-15')
```

All other partitions are excluded from the plan entirely.

### When to Partition

Partitioning is beneficial for tables over 100GB, time-series data with age-based queries, data with natural categories (regions, tenants), and scenarios where old data needs to be dropped quickly.

It is a poor fit for small tables, queries that always span all partitions, and highly random access patterns.

## Common Pitfalls

1. **Partitioning too early** - Tables under a few gigabytes rarely benefit from partitioning. The overhead of managing partitions outweighs the gains.
2. **Choosing the wrong partition key** - If most queries do not filter by the partition key, partition pruning never kicks in and you pay overhead for no benefit.
3. **Too many partitions** - Thousands of partitions increase planning time and memory usage. Monthly partitions are usually sufficient; daily partitions are rarely needed.

## Best Practices

1. **Start with range partitioning for time-series data** - Monthly partitions are the most common and effective choice for time-based data.
2. **Always include the partition key in queries** - Without a filter on the partition key, the planner must scan all partitions, negating the benefit.
3. **Create a default partition** - A default partition catches rows that do not match any defined partition, preventing INSERT failures.

## Summary

- Partitioning splits one logical table into multiple physical tables for performance and maintenance benefits.
- Range, list, and hash are the three partitioning methods.
- Partition pruning is the primary performance benefit, skipping irrelevant partitions at query time.
- Partitioning is most effective for large tables (100GB+) with time-series or categorized data.
- Choose the partition key based on your most common query patterns.

## Code Examples

**Creating a range-partitioned table and demonstrating how partition pruning skips irrelevant partitions**

```sql
-- Create a range-partitioned table by month
CREATE TABLE orders (
    id SERIAL,
    user_id INTEGER,
    total NUMERIC,
    created_at TIMESTAMP
) PARTITION BY RANGE (created_at);

-- Monthly partitions
CREATE TABLE orders_2024_01 PARTITION OF orders
    FOR VALUES FROM ('2024-01-01') TO ('2024-02-01');
CREATE TABLE orders_2024_02 PARTITION OF orders
    FOR VALUES FROM ('2024-02-01') TO ('2024-03-01');

-- Partition pruning: only orders_2024_01 is scanned
EXPLAIN SELECT * FROM orders WHERE created_at = '2024-01-15';
```


## Resources

- [Table Partitioning](https://www.postgresql.org/docs/18/ddl-partitioning.html) — Complete partitioning guide

---

> 📘 *This lesson is part of the [PostgreSQL Performance Engineering](https://stanza.dev/courses/postgresql-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*