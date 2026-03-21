---
source_course: "postgresql-performance"
source_lesson: "postgresql-performance-specialized-indexes"
---

# Specialized Index Types

## Introduction

Beyond B-tree, PostgreSQL offers specialized index types designed for specific data patterns. GIN handles arrays and JSONB, GiST manages geometric and range data, BRIN provides tiny indexes for massive ordered tables, and Bloom indexes support multi-column equality searches. Choosing the right index type for your data can provide orders-of-magnitude performance improvements.

## Key Concepts

- **GIN (Generalized Inverted Index)**: Maps each element (array item, JSON key, text token) to the rows containing it.
- **GiST (Generalized Search Tree)**: Supports spatial queries, range overlap, and nearest-neighbor searches.
- **BRIN (Block Range Index)**: Stores min/max values per block range - extremely small but requires physically ordered data.
- **Bloom index**: Uses a space-efficient probabilistic structure for multi-column equality searches.

## Real World Context

If you store JSONB documents and need to query by nested fields, a B-tree index on each field is impractical. A single GIN index handles arbitrary containment queries. For time-series data with billions of rows, a BRIN index can be 10,000x smaller than a B-tree while still providing excellent query performance.

## Deep Dive

### GIN (Generalized Inverted Index)

GIN excels at containment queries on arrays, JSONB, and full-text search:

```sql
-- JSONB index
CREATE INDEX idx_data_gin ON documents USING GIN (data);

-- Query using containment
SELECT * FROM documents WHERE data @> '{"type": "article"}';

-- Array index
CREATE INDEX idx_tags_gin ON articles USING GIN (tags);

-- Query array containment
SELECT * FROM articles WHERE tags @> ARRAY['postgresql'];

-- Full-text search
CREATE INDEX idx_content_fts ON articles 
USING GIN (to_tsvector('english', content));
```

GIN indexes are larger than B-tree and slower to update because each row may generate many index entries. PostgreSQL 18 introduces parallel GIN index creation, significantly reducing build time for large tables:

```sql
-- PG18: Parallel GIN index build
SET max_parallel_maintenance_workers = 4;
CREATE INDEX idx_data_gin ON documents USING GIN (data);
```

This leverages multiple CPU cores to build the inverted index concurrently.

### GiST (Generalized Search Tree)

GiST is the go-to index for geometric data and range types:

```sql
-- Geometric index
CREATE INDEX idx_locations_gist ON locations USING GiST (geom);

-- Range type index
CREATE INDEX idx_reservations_gist 
ON reservations USING GiST (during);

-- Query overlapping ranges
SELECT * FROM reservations 
WHERE during && '[2024-01-01, 2024-01-31]'::daterange;
```

GiST supports operators like overlap (&&), contains (@>), nearest-neighbor (ORDER BY <-> point), and more.

### BRIN (Block Range Index)

BRIN is ideal for very large tables where data is physically ordered by the indexed column:

```sql
-- Perfect for time-series data
CREATE INDEX idx_logs_brin ON logs USING BRIN (created_at);

-- Size comparison on 100M row table:
-- B-tree: 2.1 GB
-- BRIN:   128 KB (!)
```

BRIN stores min/max values per block range (128 pages by default). It requires data to be physically sorted by the indexed column. If data is scattered, BRIN performs poorly because every block range matches.

### Bloom Index

Bloom indexes use a probabilistic structure for multi-column equality searches:

```sql
-- Enable extension
CREATE EXTENSION bloom;

-- Create bloom index
CREATE INDEX idx_products_bloom ON products 
USING bloom (col1, col2, col3, col4, col5, col6);
```

Bloom indexes are useful when queries use various combinations of many columns and creating a B-tree for each combination would be impractical.

### Index Type Comparison

| Type | Best For | Size | Update Speed |
|------|----------|------|-------------|
| B-tree | General purpose, ordering | Medium | Fast |
| GIN | Arrays, JSONB, text search | Large | Slow |
| GiST | Geometry, ranges | Medium | Medium |
| BRIN | Huge tables, ordered data | Tiny | Fast |
| Bloom | Multi-column equality | Small | Fast |

## Common Pitfalls

1. **Using GIN on frequently updated tables without fastupdate** - GIN updates are expensive. Consider `WITH (fastupdate = on)` (the default) which batches updates, but be aware it can slow down reads temporarily.
2. **Creating BRIN indexes on randomly ordered data** - BRIN only works well when data is physically correlated with the indexed column. Inserting data out of order makes BRIN useless.
3. **Forgetting to install extensions** - Bloom indexes require `CREATE EXTENSION bloom` and full-text GIN indexes need the `pg_trgm` extension for trigram matching.

## Best Practices

1. **Match index type to data pattern** - Use GIN for containment queries, GiST for spatial/range data, BRIN for large naturally-ordered tables.
2. **Monitor index sizes** - GIN indexes can become very large. Check `pg_relation_size()` regularly and consider partial GIN indexes if only a subset of data is queried.
3. **Leverage PG18 parallel GIN builds** - For large tables, set `max_parallel_maintenance_workers` before creating GIN indexes to significantly reduce build time.

## Summary

- GIN indexes handle arrays, JSONB, and full-text search via inverted index structures.
- GiST indexes support spatial queries, range types, and nearest-neighbor searches.
- BRIN indexes are extremely small but require physically ordered data to be effective.
- Bloom indexes efficiently handle multi-column equality searches with minimal space.
- PostgreSQL 18 adds parallel GIN index creation for faster builds on large tables.

## Code Examples

**Creating GIN and BRIN indexes for different data patterns, including PG18 parallel GIN builds**

```sql
-- GIN index for JSONB containment queries
CREATE INDEX idx_data_gin ON documents USING GIN (data);
SELECT * FROM documents WHERE data @> '{"type": "article"}';

-- BRIN index for time-series data (128KB vs 2GB B-tree)
CREATE INDEX idx_logs_brin ON logs USING BRIN (created_at);

-- PG18: Parallel GIN index build
SET max_parallel_maintenance_workers = 4;
CREATE INDEX idx_tags_gin ON articles USING GIN (tags);
```


## Resources

- [Index Types](https://www.postgresql.org/docs/18/indexes-types.html) — All PostgreSQL index types including GIN, GiST, BRIN, and Bloom

---

> 📘 *This lesson is part of the [PostgreSQL Performance Engineering](https://stanza.dev/courses/postgresql-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*