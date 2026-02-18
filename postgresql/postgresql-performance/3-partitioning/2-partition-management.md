---
source_course: "postgresql-performance"
source_lesson: "postgresql-performance-partition-management"
---

# Managing Partitions

Effective partition management is key to long-term success.

## Adding New Partitions

```sql
-- Add partition for new month
CREATE TABLE orders_2024_03 PARTITION OF orders
    FOR VALUES FROM ('2024-03-01') TO ('2024-04-01');
```

## Default Partition

Catch rows that don't match any partition:

```sql
CREATE TABLE orders_default PARTITION OF orders DEFAULT;

-- Now any date not covered goes here
INSERT INTO orders (created_at) VALUES ('2030-01-01');
-- Stored in orders_default
```

**Warning:** Large default partitions hurt performance!

## Detaching and Dropping Partitions

```sql
-- Detach (keeps the table, removes from partition set)
ALTER TABLE orders DETACH PARTITION orders_2023_01;

-- Now it's a standalone table - can archive or drop
DROP TABLE orders_2023_01;

-- Concurrent detach (doesn't block queries)
ALTER TABLE orders DETACH PARTITION orders_2023_02 CONCURRENTLY;
```

## Attaching Existing Tables

```sql
-- Create table separately
CREATE TABLE orders_2024_04 (LIKE orders INCLUDING ALL);

-- Add CHECK constraint matching partition bounds
ALTER TABLE orders_2024_04 ADD CONSTRAINT check_date
    CHECK (created_at >= '2024-04-01' AND created_at < '2024-05-01');

-- Attach to partition set
ALTER TABLE orders 
    ATTACH PARTITION orders_2024_04
    FOR VALUES FROM ('2024-04-01') TO ('2024-05-01');
```

**Note:** Pre-adding the CHECK constraint avoids a full table scan during ATTACH.

## Indexes on Partitioned Tables

```sql
-- Create index on partitioned table
CREATE INDEX idx_orders_user ON orders(user_id);

-- PostgreSQL automatically creates matching indexes on all partitions!
```

## Automating Partition Creation

```sql
-- Function to create monthly partitions
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

-- Create next 6 months
SELECT create_monthly_partition('orders', 2024, generate_series(1, 6));
```

## Partition-wise Operations

```sql
-- Enable partition-wise joins
SET enable_partitionwise_join = on;

-- Enable partition-wise aggregates
SET enable_partitionwise_aggregate = on;
```

These settings allow PostgreSQL to process partitions in parallel.

📖 [Partition Maintenance](https://www.postgresql.org/docs/18/ddl-partitioning.html#DDL-PARTITIONING-IMPLEMENTATION)

---

> 📘 *This lesson is part of the [PostgreSQL Performance Engineering](https://stanza.dev/courses/postgresql-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*