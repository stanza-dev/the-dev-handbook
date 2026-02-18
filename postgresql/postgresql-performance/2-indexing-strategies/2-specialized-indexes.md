---
source_course: "postgresql-performance"
source_lesson: "postgresql-performance-specialized-indexes"
---

# Specialized Index Types

Beyond B-tree, PostgreSQL offers specialized indexes for specific use cases.

## GIN (Generalized Inverted Index)

Best for: **Arrays, JSONB, Full-text search**

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

**Characteristics:**
- Handles "contains" queries efficiently
- Larger than B-tree indexes
- Slower to update (many index entries per row)

## GiST (Generalized Search Tree)

Best for: **Geometric data, Range types, Full-text**

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

## BRIN (Block Range Index)

Best for: **Very large tables with naturally ordered data**

```sql
-- Perfect for time-series data
CREATE INDEX idx_logs_brin ON logs USING BRIN (created_at);

-- Size comparison on 100M row table:
-- B-tree: 2.1 GB
-- BRIN:   128 KB (!)
```

**How BRIN works:**
- Stores min/max values per block range (128 pages default)
- Extremely small index size
- Requires data to be physically sorted by indexed column

**Use when:**
- Table is very large (millions+ rows)
- Data is append-only or naturally ordered
- Slight imprecision is acceptable

## Bloom Index

Best for: **Multi-column equality searches on any column combination**

```sql
-- Enable extension
CREATE EXTENSION bloom;

-- Create bloom index
CREATE INDEX idx_products_bloom ON products 
USING bloom (col1, col2, col3, col4, col5, col6);
```

**Use when:**
- Many columns, queries use various combinations
- B-tree would require many separate indexes

## Index Type Comparison

| Type | Best For | Size | Update Speed |
|------|----------|------|-------------|
| B-tree | General purpose, ordering | Medium | Fast |
| GIN | Arrays, JSONB, text search | Large | Slow |
| GiST | Geometry, ranges | Medium | Medium |
| BRIN | Huge tables, ordered data | Tiny | Fast |
| Bloom | Multi-column equality | Small | Fast |

📖 [Index Types](https://www.postgresql.org/docs/18/indexes-types.html)

---

> 📘 *This lesson is part of the [PostgreSQL Performance Engineering](https://stanza.dev/courses/postgresql-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*