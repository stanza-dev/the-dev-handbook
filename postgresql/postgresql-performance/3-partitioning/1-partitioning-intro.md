---
source_course: "postgresql-performance"
source_lesson: "postgresql-performance-partitioning-intro"
---

# Table Partitioning

Partitioning divides one logical table into multiple physical tables for better performance and easier maintenance.

## Why Partition?

**Performance:**
- **Partition pruning**: Only scan relevant partitions
- **Parallel operations**: Process partitions simultaneously
- **Smaller indexes**: Each partition has smaller indexes

**Maintenance:**
- **Drop old data instantly**: Detach and drop partition
- **Vacuum individual partitions**: Less lock time
- **Archive separately**: Move old partitions to slower storage

## Partitioning Methods

### Range Partitioning
Divide by value ranges (dates, numbers):

```sql
CREATE TABLE orders (
    id SERIAL,
    user_id INTEGER,
    total NUMERIC,
    created_at TIMESTAMP
) PARTITION BY RANGE (created_at);

-- Create partitions for each month
CREATE TABLE orders_2024_01 PARTITION OF orders
    FOR VALUES FROM ('2024-01-01') TO ('2024-02-01');

CREATE TABLE orders_2024_02 PARTITION OF orders
    FOR VALUES FROM ('2024-02-01') TO ('2024-03-01');
```

### List Partitioning
Divide by specific values:

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

### Hash Partitioning
Distribute evenly by hash value:

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

## Partition Pruning

```sql
-- Only scans relevant partition(s)
EXPLAIN SELECT * FROM orders WHERE created_at = '2024-01-15';
```

```
Append  (cost=0.00..25.88 rows=6 width=44)
  ->  Seq Scan on orders_2024_01  (cost=0.00..25.88 rows=6 width=44)
        Filter: (created_at = '2024-01-15')
```

Other partitions are **not scanned at all**!

## When to Partition

✅ **Good candidates:**
- Tables > 100GB
- Time-series data with age-based queries
- Data with natural categories (regions, tenants)
- Need to quickly drop old data

❌ **Poor candidates:**
- Small tables
- Queries that span all partitions
- Highly random access patterns

📖 [Table Partitioning](https://www.postgresql.org/docs/18/ddl-partitioning.html)

## Resources

- [Table Partitioning](https://www.postgresql.org/docs/18/ddl-partitioning.html) — Complete partitioning guide

---

> 📘 *This lesson is part of the [PostgreSQL Performance Engineering](https://stanza.dev/courses/postgresql-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*