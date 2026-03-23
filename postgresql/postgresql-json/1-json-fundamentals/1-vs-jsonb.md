---
source_course: "postgresql-json"
source_lesson: "postgresql-json-json-vs-jsonb"
---

# JSON vs JSONB: Understanding the Difference

## Introduction
PostgreSQL provides two JSON data types: `json` and `jsonb`. While both store JSON data, they differ significantly in storage, performance, and functionality. Understanding when to use each type is essential for building efficient database schemas.

## Key Concepts
- **json**: A data type that stores an exact copy of the JSON input text, preserving whitespace, key order, and duplicate keys.
- **jsonb**: A data type that stores JSON in a decomposed binary format, enabling indexing, faster processing, and additional operators.
- **GIN Index**: A Generalized Inverted Index that can be created on `jsonb` columns to accelerate containment and existence queries.
- **Key Deduplication**: When converting to `jsonb`, duplicate keys are removed and only the last value for each key is retained.

## Real World Context
Most web applications store semi-structured data such as user preferences, API payloads, or product attributes. Choosing between `json` and `jsonb` determines query performance, storage efficiency, and available operations. Companies like Shopify and GitHub use JSONB columns extensively for flexible attribute storage alongside relational columns.

## Deep Dive

### The json Type

The `json` type stores an exact copy of the input text. This means it preserves whitespace, key ordering, and even duplicate keys:

```sql
SELECT '{"bar": "baz", "balance": 7.77, "active":false}'::json;
-- Preserves whitespace and key order exactly
```

This exact preservation makes `json` suitable for audit trails and logging where you need to store the original payload without modification.

**Characteristics of json:**
- Stores exact input text
- Preserves whitespace and key order
- Preserves duplicate keys
- Must reparse on each operation
- Cannot be indexed (except expression indexes)

### The jsonb Type

The `jsonb` type stores data in a decomposed binary format that is pre-parsed and optimized for querying:

```sql
SELECT '{"bar": "baz", "balance": 7.77, "active":false}'::jsonb;
-- Output: {"bar": "baz", "active": false, "balance": 7.77}
-- Keys reordered, whitespace normalized
```

Because `jsonb` is pre-parsed, every subsequent operation is faster since no reparsing is needed.

**Characteristics of jsonb:**
- Binary storage format
- Removes whitespace, reorders keys
- Removes duplicate keys (last value wins)
- Faster to process (pre-parsed)
- Supports GIN indexing
- Supports additional operators like `@>`, `?`, `?|`, `?&`

### When to Use Each

| Use Case | Recommended |
|----------|-------------|
| API response logging (exact preservation) | json |
| Queryable document storage | jsonb |
| Full-text search on JSON | jsonb |
| Configuration storage | jsonb |
| Audit trails (exact input) | json |

In practice, `jsonb` is the right choice for approximately 95% of use cases. Only reach for `json` when you need exact text preservation.

## Common Pitfalls
1. **Using json instead of jsonb for queryable data** — The `json` type requires reparsing on every access and cannot use GIN indexes, leading to slow queries on large tables.
2. **Assuming duplicate keys are preserved in jsonb** — JSONB silently discards duplicate keys and keeps only the last value, which can cause data loss if your input relies on duplicates.
3. **Storing large documents without considering column size** — Both types support up to 1 GB per value, but very large JSON documents degrade query performance.

## Best Practices
1. **Default to jsonb** — Unless you have a specific requirement for exact text preservation, always use `jsonb` for its indexing and performance advantages.
2. **Add a CHECK constraint for type validation** — Use `CHECK (jsonb_typeof(column) = 'object')` to ensure the column always contains a JSON object rather than an array or scalar.
3. **Combine with relational columns** — Store frequently queried or constrained fields as regular columns and use JSONB for flexible, variable attributes.

## Summary
- PostgreSQL offers two JSON types: `json` (exact text) and `jsonb` (binary, pre-parsed).
- `jsonb` supports GIN indexes, containment operators, and is faster for querying.
- `json` preserves whitespace, key order, and duplicate keys.
- Use `jsonb` in 95% of cases; reserve `json` for audit logging or exact-text requirements.
- JSONB deduplicates keys, keeping only the last value for each key.

## Code Examples

**JSON vs JSONB Comparison**

```sql
-- JSON preserves exact input
SELECT '{"a":1,"a":2}'::json;  -- {"a":1,"a":2} (both keys kept)

-- JSONB deduplicates keys
SELECT '{"a":1,"a":2}'::jsonb;  -- {"a": 2} (last value wins)

-- Performance difference
CREATE TABLE json_test (data json);
CREATE TABLE jsonb_test (data jsonb);

-- JSONB can be indexed
CREATE INDEX ON jsonb_test USING GIN (data);
```


## Resources

- [JSON Types](https://www.postgresql.org/docs/current/datatype-json.html) — undefined

---

> 📘 *This lesson is part of the [PostgreSQL JSON & Document Processing](https://stanza.dev/courses/postgresql-json) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*