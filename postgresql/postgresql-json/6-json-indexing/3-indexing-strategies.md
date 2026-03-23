---
source_course: "postgresql-json"
source_lesson: "postgresql-json-indexing-strategies"
---

# Combined Indexing Strategies

## Introduction
Real-world applications rarely use a single index type. The best JSONB performance comes from combining GIN indexes for flexible queries, expression indexes for specific field lookups, and partial indexes for selective filtering. This lesson covers strategies for choosing and combining index types.

## Key Concepts
- **Partial index**: An index that includes only rows matching a WHERE condition, reducing size and improving write performance.
- **Covering index (INCLUDE)**: A B-tree index that stores additional columns in the leaf pages, enabling index-only scans.
- **Index-only scan**: A query that reads all needed data from the index itself without accessing the heap (table).
- **Generated columns**: Computed columns that extract and store JSONB values, enabling standard B-tree indexes without expression syntax.

## Real World Context
A high-traffic e-commerce platform might use: a GIN index for flexible product filtering by arbitrary attributes, an expression index on `(data->>'status')` for the hot path that checks order status, and a partial GIN index on `WHERE data->>'active' = 'true'` to exclude archived data. Understanding when to combine these approaches is what separates good performance from great performance.

## Deep Dive

### Partial GIN Indexes

Partial indexes only include rows matching a condition, making them smaller and faster:

```sql
-- Only index active products
CREATE INDEX idx_active_products_attrs 
    ON products USING GIN (attributes)
    WHERE active = true;

-- Only index recent events
CREATE INDEX idx_recent_events_data 
    ON events USING GIN (data)
    WHERE created_at > '2024-01-01';
```

Partial indexes are smaller, update less frequently, and query faster because they skip irrelevant rows.

### GIN + Expression: Complementary Indexes

Use both index types on the same column for different query patterns:

```sql
-- GIN for containment queries
CREATE INDEX idx_events_gin ON events USING GIN (data);

-- Expression for specific field equality
CREATE INDEX idx_events_type ON events ((data->>'type'));
CREATE INDEX idx_events_user ON events ((data->>'user_id'));

-- Each index serves different queries:
-- Uses GIN
SELECT * FROM events WHERE data @> '{"source": "mobile"}';

-- Uses expression index
SELECT * FROM events WHERE data->>'type' = 'purchase';
```

This combination covers both flexible filtering and specific field lookups.

### Generated Columns as an Alternative

For heavily queried fields, generated columns are cleaner than expression indexes:

```sql
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    data JSONB NOT NULL,
    -- Generated columns
    status TEXT GENERATED ALWAYS AS (data->>'status') STORED,
    customer_id TEXT GENERATED ALWAYS AS (data->>'customer_id') STORED
);

-- Standard B-tree indexes
CREATE INDEX idx_orders_status ON orders (status);
CREATE INDEX idx_orders_customer ON orders (customer_id);
```

Generated columns are automatically kept in sync and allow standard index syntax.

### Choosing the Right Strategy

| Scenario | Index Type |
|----------|------------|
| Flexible attribute filtering | GIN (jsonb_ops) |
| Containment queries only | GIN (jsonb_path_ops) |
| Specific field equality | Expression index |
| Numeric range on JSON field | Expression index with cast |
| Many fields need indexing | Generated columns + B-tree |
| Large table, few active rows | Partial index |

## Common Pitfalls
1. **Over-indexing** — Each index slows down writes. Only create indexes for queries that actually need them.
2. **Not verifying index usage** — Always run `EXPLAIN ANALYZE` to confirm your index is being used.
3. **Ignoring partial indexes** — If most queries filter by a common condition (e.g., active = true), a partial index is much more efficient.

## Best Practices
1. **Start with GIN, add expression indexes for hot paths** — GIN covers most queries; add targeted expression indexes for the most frequent specific-field lookups.
2. **Use EXPLAIN ANALYZE to validate** — After creating indexes, verify they are used with `EXPLAIN ANALYZE`.
3. **Monitor with pg_stat_user_indexes** — Check `idx_scan` to see how often each index is used and remove unused ones.

## Summary
- Combine GIN indexes (flexible queries) with expression indexes (specific field lookups) for comprehensive coverage.
- Partial indexes reduce size and improve performance by indexing only relevant rows.
- Generated columns are a clean alternative to expression indexes for heavily queried fields.
- Always validate index usage with EXPLAIN ANALYZE.
- Monitor index usage statistics and remove unused indexes to reduce write overhead.

## Code Examples

**Combined Indexing Strategy**

```sql
-- GIN for flexible queries
CREATE INDEX idx_products_gin ON products USING GIN (attributes jsonb_path_ops);

-- Expression for hot-path lookups
CREATE INDEX idx_products_brand ON products ((attributes->>'brand'));

-- Partial index for active-only queries
CREATE INDEX idx_active_products ON products USING GIN (attributes)
    WHERE is_active = true;

-- Verify each index is used
EXPLAIN ANALYZE SELECT * FROM products WHERE attributes @> '{"color": "red"}';
EXPLAIN ANALYZE SELECT * FROM products WHERE attributes->>'brand' = 'Apple';

-- Monitor index usage
SELECT indexrelname, idx_scan, idx_tup_read
FROM pg_stat_user_indexes
WHERE schemaname = 'public'
ORDER BY idx_scan DESC;
```


## Resources

- [Partial Indexes](https://www.postgresql.org/docs/current/indexes-partial.html) — undefined

---

> 📘 *This lesson is part of the [PostgreSQL JSON & Document Processing](https://stanza.dev/courses/postgresql-json) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*