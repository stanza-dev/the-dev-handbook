---
source_course: "postgresql-performance"
source_lesson: "postgresql-performance-partition-management"
---

# Managing Partitions

## Introduction

Creating partitions is only the beginning. Effective partition management involves adding new partitions ahead of time, safely removing old data, automating the lifecycle, and handling edge cases like default partitions. Poor management can lead to INSERT failures, bloated default partitions, or downtime during maintenance.

## Key Concepts

- **Default partition**: A catch-all partition for rows that do not match any defined range or list.
- **DETACH PARTITION**: Removes a partition from the partitioned table but keeps it as a standalone table.
- **ATTACH PARTITION**: Adds an existing table as a partition.
- **CONCURRENTLY**: A modifier that allows detaching without blocking queries.
- **Partition-wise operations**: Settings that allow the planner to process partitions in parallel.

## Real World Context

In production systems, you need automated scripts that create next month's partition before the month starts, and archive or drop old partitions on a schedule. Without automation, you risk INSERT failures when data arrives for a period with no matching partition.

## Deep Dive

### Adding New Partitions

Create partitions proactively before data arrives:

```sql
CREATE TABLE orders_2024_03 PARTITION OF orders
    FOR VALUES FROM ('2024-03-01') TO ('2024-04-01');
```

If a partition does not exist for incoming data and there is no default partition, the INSERT will fail with an error.

### Default Partition

A default partition catches rows that do not match any defined partition:

```sql
CREATE TABLE orders_default PARTITION OF orders DEFAULT;
```

This prevents INSERT failures, but large default partitions hurt performance because they must be scanned when pruning cannot eliminate them. Monitor and split defaults regularly.

### Detaching and Dropping Partitions

Detaching removes a partition from the set while keeping the data:

```sql
-- Detach (keeps the table, removes from partition set)
ALTER TABLE orders DETACH PARTITION orders_2023_01;

-- Now it's a standalone table — can archive or drop
DROP TABLE orders_2023_01;

-- Concurrent detach (doesn't block queries)
ALTER TABLE orders DETACH PARTITION orders_2023_02 CONCURRENTLY;
```

Concurrent detach is preferred in production because it does not block ongoing queries.

### Attaching Existing Tables

You can prepare a table separately and then attach it:

```sql
CREATE TABLE orders_2024_04 (LIKE orders INCLUDING ALL);

-- Add CHECK constraint matching partition bounds
ALTER TABLE orders_2024_04 ADD CONSTRAINT check_date
    CHECK (created_at >= '2024-04-01' AND created_at < '2024-05-01');

-- Attach to partition set
ALTER TABLE orders 
    ATTACH PARTITION orders_2024_04
    FOR VALUES FROM ('2024-04-01') TO ('2024-05-01');
```

Pre-adding the CHECK constraint avoids a full table scan during ATTACH, which is important for large tables.

### Indexes on Partitioned Tables

Creating an index on the partitioned table automatically creates matching indexes on all partitions:

```sql
CREATE INDEX idx_orders_user ON orders(user_id);
-- PostgreSQL creates idx on each partition automatically
```

This simplifies index management across many partitions.

### Automating Partition Creation

A function to create monthly partitions programmatically:

```sql
CREATE OR REPLACE FUNCTION create_monthly_partition(
    table_name TEXT,
    year INT,
    month INT
) RETURNS VOID AS $$
DECLARE
    partition_name TEXT;
    start_date DATE;
    end_date DATE;
BEGIN
    partition_name := table_name || '_' || year || '_' || LPAD(month::TEXT, 2, '0');
    start_date := make_date(year, month, 1);
    end_date := start_date + INTERVAL '1 month';
    
    EXECUTE format(
        'CREATE TABLE IF NOT EXISTS %I PARTITION OF %I FOR VALUES FROM (%L) TO (%L)',
        partition_name, table_name, start_date, end_date
    );
END;
$$ LANGUAGE plpgsql;

SELECT create_monthly_partition('orders', 2024, generate_series(1, 6));
```

This function creates partitions ahead of time, preventing INSERT failures.

### Partition-wise Operations

Enable partition-wise joins and aggregates for better parallel performance:

```sql
SET enable_partitionwise_join = on;
SET enable_partitionwise_aggregate = on;
```

These settings allow PostgreSQL to join and aggregate within each partition independently, enabling parallelism.

## Common Pitfalls

1. **Forgetting to create future partitions** - Without a default partition or proactive creation, INSERTs for new time periods fail.
2. **Letting the default partition grow** - A large default partition defeats the purpose of partitioning because it must be scanned alongside targeted partitions.
3. **Using DETACH without CONCURRENTLY in production** - Non-concurrent detach takes an ACCESS EXCLUSIVE lock, blocking all queries on the table.

## Best Practices

1. **Automate partition creation** - Use a cron job or pg_cron to create partitions at least one period ahead of time.
2. **Always create a default partition** - It acts as a safety net for unexpected data, preventing INSERT failures.
3. **Use CONCURRENTLY for production detach** - `DETACH PARTITION ... CONCURRENTLY` avoids blocking ongoing queries.

## Summary

- Create partitions proactively to avoid INSERT failures.
- Default partitions catch unmatched rows but should be kept small.
- DETACH PARTITION keeps data while removing it from the partition set.
- Use CONCURRENTLY for detaching in production to avoid blocking queries.
- Automate partition lifecycle with functions or pg_cron.

## Code Examples

**Detaching old partitions concurrently and attaching pre-built tables with CHECK constraints to avoid full scans**

```sql
-- Safely remove old data: detach then drop
ALTER TABLE orders DETACH PARTITION orders_2023_01 CONCURRENTLY;
DROP TABLE orders_2023_01;

-- Attach a pre-built table with constraint to avoid full scan
CREATE TABLE orders_2024_04 (LIKE orders INCLUDING ALL);
ALTER TABLE orders_2024_04 ADD CONSTRAINT check_date
    CHECK (created_at >= '2024-04-01' AND created_at < '2024-05-01');
ALTER TABLE orders ATTACH PARTITION orders_2024_04
    FOR VALUES FROM ('2024-04-01') TO ('2024-05-01');
```


## Resources

- [Partition Maintenance](https://www.postgresql.org/docs/18/ddl-partitioning.html#DDL-PARTITIONING-DECLARATIVE-MAINTENANCE) — Partition maintenance operations

---

> 📘 *This lesson is part of the [PostgreSQL Performance Engineering](https://stanza.dev/courses/postgresql-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*