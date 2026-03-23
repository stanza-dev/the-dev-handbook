---
source_course: "postgresql-json"
source_lesson: "postgresql-json-gin-indexes"
---

# GIN Indexes for JSONB

## Introduction
GIN (Generalized Inverted Index) indexes dramatically speed up JSONB queries by indexing the internal structure of JSON documents. PostgreSQL offers two GIN operator classes for JSONB, each optimized for different query patterns. Choosing the right one can mean the difference between millisecond and second-long queries on large tables.

## Key Concepts
- **GIN (Generalized Inverted Index)**: An index type that maps individual keys and values to the rows containing them, enabling fast lookups for containment and existence queries.
- **jsonb_ops**: The default GIN operator class that supports all JSONB operators (`@>`, `?`, `?|`, `?&`, `@?`, `@@`).
- **jsonb_path_ops**: An alternative GIN operator class optimized for containment queries (`@>`, `@?`, `@@`) with a smaller index footprint.
- **Operator class**: A specification that tells PostgreSQL which operators an index supports and how to index the data.

## Real World Context
Any application storing JSONB data at scale needs GIN indexes. Without them, every query scans the entire table. E-commerce platforms with millions of products use GIN indexes on attribute columns to serve filtered product listings in milliseconds. The choice between `jsonb_ops` and `jsonb_path_ops` depends on your query patterns — if you only use `@>` containment, `jsonb_path_ops` saves significant disk space.

## Deep Dive

### jsonb_ops (Default)

The default operator class supports all JSONB operators:

```sql
CREATE INDEX idx_products_attrs ON products USING GIN (attributes);

-- Supported operators
@>   -- Containment
?    -- Key exists
?|   -- Any key exists
?&   -- All keys exist
@?   -- JSON path exists
@@   -- JSON path predicate
```

This is the most flexible choice and is appropriate when your queries use a mix of different operators.

### jsonb_path_ops

Optimized for containment queries with a smaller index size:

```sql
CREATE INDEX idx_products_attrs_path 
    ON products USING GIN (attributes jsonb_path_ops);

-- Supported operators (fewer than jsonb_ops)
@>   -- Containment
@?   -- JSON path exists
@@   -- JSON path predicate
```

The tradeoff is that `jsonb_path_ops` does not support key existence operators (`?`, `?|`, `?&`).

### Comparison

| Feature | jsonb_ops | jsonb_path_ops |
|---------|-----------|----------------|
| Index size | Larger | 2-3x smaller |
| ? operator | Yes | No |
| ?& ?| operators | Yes | No |
| @> operator | Yes | Yes |
| Best for | Diverse queries | Containment-only queries |

In benchmarks, `jsonb_path_ops` indexes can be 2-3 times smaller than `jsonb_ops` indexes on the same data.

### Query Examples

These queries benefit from GIN indexes:

```sql
-- These queries use the GIN index
SELECT * FROM products 
WHERE attributes @> '{"brand": "Apple"}';

SELECT * FROM products 
WHERE attributes ? 'warranty';  -- Only jsonb_ops

SELECT * FROM products 
WHERE attributes @? '$.specs.ram ? (@ > 16)';
```

However, extraction operators do not use GIN indexes:

```sql
-- This does NOT use GIN indexes
SELECT * FROM products 
WHERE attributes->>'brand' = 'Apple';
-- Use an expression index instead (next lesson)
```

## Common Pitfalls
1. **Expecting GIN to help with ->> queries** — The `->`, `->>`, and equality operators do NOT use GIN indexes. You need expression indexes for those.
2. **Creating both jsonb_ops and jsonb_path_ops on the same column** — This wastes space. Choose one based on your query patterns.
3. **Forgetting GIN index maintenance overhead** — GIN indexes are slower to update than B-tree. On write-heavy tables, consider partial indexes or `fastupdate = off`.

## Best Practices
1. **Start with jsonb_ops** — It covers all operators and is the safest default choice.
2. **Switch to jsonb_path_ops for containment-heavy workloads** — If your queries only use `@>` and `@?`, the smaller index size is a significant advantage.
3. **Monitor index size and usage** — Use `pg_stat_user_indexes` to verify your GIN index is actually being used by queries.

## Summary
- GIN indexes enable fast JSONB queries for containment, existence, and JSON path operations.
- `jsonb_ops` (default) supports all operators; `jsonb_path_ops` is smaller but limited to containment.
- Extraction operators (`->`, `->>`) do not use GIN indexes.
- Choose the operator class based on your query patterns.
- GIN indexes have higher write overhead than B-tree; consider this for write-heavy tables.

## Code Examples

**GIN Index Strategy**

```sql
-- Default GIN index for general queries
CREATE INDEX idx_events_data ON events USING GIN (data);

-- Optimized index for containment-heavy workloads
CREATE INDEX idx_events_data_path ON events USING GIN (data jsonb_path_ops);

-- Verify index usage
EXPLAIN ANALYZE
SELECT * FROM events 
WHERE data @> '{"type": "click", "source": "mobile"}';

-- Index NOT used for this (use expression index instead)
EXPLAIN ANALYZE
SELECT * FROM events 
WHERE data->>'type' = 'click';
```


## Resources

- [JSON Indexing](https://www.postgresql.org/docs/current/datatype-json.html#JSON-INDEXING) — undefined

---

> 📘 *This lesson is part of the [PostgreSQL JSON & Document Processing](https://stanza.dev/courses/postgresql-json) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*