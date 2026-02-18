---
source_course: "postgresql-json"
source_lesson: "json-operators"
---

# JSON Extraction Operators

PostgreSQL provides powerful operators for extracting data from JSON.

## Basic Extraction Operators

| Operator | Description | Return Type |
|----------|-------------|-------------|
| `->` | Get JSON object field by key | json/jsonb |
| `->>` | Get JSON object field as text | text |
| `#>` | Get JSON object at path | json/jsonb |
| `#>>` | Get JSON object at path as text | text |

```sql
-- Sample data
SELECT '{"name": "Alice", "address": {"city": "NYC"}}'::jsonb AS data;

-- Extract as JSON (keeps type)
SELECT data->'name' FROM ...;  -- "Alice" (jsonb)

-- Extract as text
SELECT data->>'name' FROM ...;  -- Alice (text)

-- Nested extraction
SELECT data->'address'->'city' FROM ...;  -- "NYC"
SELECT data->'address'->>'city' FROM ...;  -- NYC

-- Path extraction
SELECT data#>'{address,city}' FROM ...;  -- "NYC"
SELECT data#>>'{address,city}' FROM ...;  -- NYC
```

## Array Access

```sql
SELECT '[1, 2, 3]'::jsonb->0;  -- 1
SELECT '[1, 2, 3]'::jsonb->-1; -- 3 (last element)
SELECT '[1, 2, 3]'::jsonb->>1; -- '2' (as text)
```

## Practical Example

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

## Code Examples

```undefined
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