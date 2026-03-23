---
source_course: "postgresql-json"
source_lesson: "postgresql-json-operators"
---

# JSON Extraction Operators

## Introduction
PostgreSQL provides a family of operators for extracting data from JSON documents. These operators let you navigate into nested objects, access array elements, and extract values either as JSON or as plain text. They form the foundation of every JSON query you will write.

## Key Concepts
- **`->` operator**: Extracts a JSON object field by key or an array element by index, returning the result as `json`/`jsonb`.
- **`->>` operator**: Extracts a JSON object field by key or an array element by index, returning the result as `text`.
- **`#>` operator**: Extracts a JSON value at a specified path (array of keys), returning `json`/`jsonb`.
- **`#>>` operator**: Extracts a JSON value at a specified path, returning `text`.

## Real World Context
Every application that reads JSON data from PostgreSQL uses these operators. Whether you are building a dashboard that displays user profile fields, filtering products by nested attributes, or extracting timestamps from event payloads, extraction operators are the starting point. They appear in SELECT lists, WHERE clauses, and JOIN conditions throughout production codebases.

## Deep Dive

### Basic Extraction Operators

The four core operators form a 2x2 matrix of choices: extract by key/index or by path, and return JSON or text.

| Operator | Description | Return Type |
|----------|-------------|-------------|
| `->` | Get JSON object field by key | json/jsonb |
| `->>` | Get JSON object field as text | text |
| `#>` | Get JSON object at path | json/jsonb |
| `#>>` | Get JSON object at path as text | text |

Here is how they work in practice:

```sql
-- Sample data
SELECT '{"name": "Alice", "address": {"city": "NYC"}}'::jsonb AS data;

-- Extract as JSON (keeps type)
SELECT data->'name' FROM ...;  -- "Alice" (jsonb)

-- Extract as text
SELECT data->>'name' FROM ...;  -- Alice (text)

-- Nested extraction by chaining
SELECT data->'address'->'city' FROM ...;  -- "NYC"
SELECT data->'address'->>'city' FROM ...;  -- NYC

-- Path extraction in one step
SELECT data#>'{address,city}' FROM ...;  -- "NYC"
SELECT data#>>'{address,city}' FROM ...;  -- NYC
```

The path operators (`#>` and `#>>`) are especially useful for deeply nested structures where chaining multiple `->` operators becomes unwieldy.

### Array Access

The `->` and `->>` operators also work with arrays using integer indices:

```sql
SELECT '[1, 2, 3]'::jsonb->0;  -- 1
SELECT '[1, 2, 3]'::jsonb->-1; -- 3 (last element)
SELECT '[1, 2, 3]'::jsonb->>1; -- '2' (as text)
```

Negative indices count from the end of the array, which is convenient for accessing the last element.

### Practical Example

Here is a complete example showing extraction in a real query:

```sql
CREATE TABLE events (
    id SERIAL PRIMARY KEY,
    data JSONB
);

INSERT INTO events (data) VALUES
('{"type": "click", "user": {"id": 1, "name": "Alice"}, "timestamp": "2024-01-15"}');

-- Query with operators
SELECT 
    data->>'type' AS event_type,
    data->'user'->>'name' AS user_name,
    data->>'timestamp' AS event_time
FROM events
WHERE data->>'type' = 'click';
```

Notice that `->>` is used in the WHERE clause because we need to compare against a text value.

## Common Pitfalls
1. **Comparing with -> instead of ->>** — `data->'name' = 'Alice'` fails because `->` returns jsonb, not text. Use `data->>'name' = 'Alice'` or `data->'name' = '"Alice"'::jsonb`.
2. **Forgetting that -> returns NULL for missing keys** — Accessing a non-existent key returns NULL silently rather than raising an error, which can hide bugs.
3. **Using extraction operators in WHERE without indexes** — `WHERE data->>'type' = 'click'` does a full table scan unless you create an expression index on `(data->>'type')`.

## Best Practices
1. **Use ->> for comparisons and display** — Always use the text-returning operator when comparing values in WHERE clauses or displaying results.
2. **Use -> for further JSON navigation** — When you need to chain into nested objects, use `->` to keep the JSON type for continued navigation.
3. **Create expression indexes for frequently filtered fields** — If you often filter on `data->>'type'`, create `CREATE INDEX ON events ((data->>'type'))` for fast lookups.

## Summary
- `->` and `->>` extract by key or index, returning JSON or text respectively.
- `#>` and `#>>` extract by path, useful for deeply nested access.
- Negative array indices count from the end.
- Use `->>` (text) for WHERE clause comparisons and `->` (JSON) for further navigation.
- Extraction operators do not use GIN indexes; create expression indexes for filtered fields.

## Code Examples

**JSON Extraction Examples**

```sql
-- Complex nested extraction
SELECT 
    p.name,
    p.attributes->>'brand' AS brand,
    p.attributes->'specs'->>'ram' AS ram_gb,
    p.attributes#>>'{specs,storage}' AS storage_gb
FROM products p
WHERE p.attributes->'specs' IS NOT NULL;

-- Array element access
SELECT 
    data->>'name' AS name,
    data->'tags'->0 AS first_tag,
    data->'tags'->-1 AS last_tag
FROM items;
```


## Resources

- [JSON Operators](https://www.postgresql.org/docs/current/functions-json.html#FUNCTIONS-JSON-OP-TABLE) — undefined

---

> 📘 *This lesson is part of the [PostgreSQL JSON & Document Processing](https://stanza.dev/courses/postgresql-json) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*