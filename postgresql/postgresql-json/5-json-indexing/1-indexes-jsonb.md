---
source_course: "postgresql-json"
source_lesson: "gin-indexes-jsonb"
---

# GIN Indexes for JSONB

GIN (Generalized Inverted Index) indexes dramatically speed up JSONB queries. PostgreSQL offers two GIN operator classes for JSONB.

## jsonb_ops (Default)

Supports all JSONB operators:

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

## jsonb_path_ops

Optimized for containment queries, smaller index size:

```sql
CREATE INDEX idx_products_attrs_path 
    ON products USING GIN (attributes jsonb_path_ops);

-- Supported operators (fewer than jsonb_ops)
@>   -- Containment
@?   -- JSON path exists
@@   -- JSON path predicate
```

## Comparison

| Feature | jsonb_ops | jsonb_path_ops |
|---------|-----------|----------------|
| Index size | Larger | Smaller |
| ? operator | Yes | No |
| ?& ?| operators | Yes | No |
| @> operator | Yes | Yes |
| Best for | Diverse queries | Containment queries |

## Query Examples

```sql
-- These queries use the GIN index
SELECT * FROM products 
WHERE attributes @> '{"brand": "Apple"}';

SELECT * FROM products 
WHERE attributes ? 'warranty';  -- Only jsonb_ops

SELECT * FROM products 
WHERE attributes @? '$.specs.ram ? (@ > 16)';
```

**Important:** The `->`, `->>`, and equality operators do NOT use GIN indexes!

## Code Examples

```undefined
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