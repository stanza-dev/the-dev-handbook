---
source_course: "postgresql-json"
source_lesson: "json-vs-jsonb"
---

# JSON vs JSONB: Understanding the Difference

PostgreSQL provides two JSON data types: `json` and `jsonb`. While both store JSON data, they differ significantly in storage, performance, and functionality.

## JSON Type

The `json` type stores an exact copy of the input text:

```sql
SELECT '{"bar": "baz", "balance": 7.77, "active":false}'::json;
-- Preserves whitespace and key order exactly
```

**Characteristics:**
- Stores exact input text
- Preserves whitespace and key order
- Preserves duplicate keys
- Must reparse on each operation
- Cannot be indexed (except expression indexes)

## JSONB Type

The `jsonb` type stores data in a decomposed binary format:

```sql
SELECT '{"bar": "baz", "balance": 7.77, "active":false}'::jsonb;
-- Output: {"bar": "baz", "active": false, "balance": 7.77}
-- Keys reordered, whitespace normalized
```

**Characteristics:**
- Binary storage format
- Removes whitespace, reorders keys
- Removes duplicate keys (last value wins)
- Faster to process (pre-parsed)
- Supports GIN indexing
- Supports additional operators

## When to Use Each

| Use Case | Recommended |
|----------|-------------|
| API response logging (exact preservation) | json |
| Queryable document storage | jsonb |
| Full-text search on JSON | jsonb |
| Configuration storage | jsonb |
| Audit trails (exact input) | json |

**Best Practice:** Use `jsonb` in 95% of cases. Only use `json` when you need exact text preservation.

## Code Examples

```undefined
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