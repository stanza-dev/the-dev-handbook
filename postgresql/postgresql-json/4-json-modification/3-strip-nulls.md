---
source_course: "postgresql-json"
source_lesson: "jsonb-strip-nulls"
---

# Cleaning JSON Data

PostgreSQL provides functions to clean and transform JSON data.

## jsonb_strip_nulls: Remove Null Values

```sql
-- Remove null keys recursively
SELECT jsonb_strip_nulls(
    '{"a": 1, "b": null, "c": {"d": null, "e": 2}}'::jsonb
);
-- Result: {"a": 1, "c": {"e": 2}}

-- Useful for API responses
UPDATE api_cache
SET response = jsonb_strip_nulls(response);
```

## jsonb_pretty: Format for Readability

```sql
SELECT jsonb_pretty('{"name":"Alice","address":{"city":"NYC"}}'::jsonb);
-- Result:
-- {
--     "name": "Alice",
--     "address": {
--         "city": "NYC"
--     }
-- }
```

## Type Conversions

```sql
-- Convert JSONB to arrays/records
SELECT * FROM jsonb_array_elements('[1, 2, 3]'::jsonb);
-- Returns rows: 1, 2, 3

SELECT * FROM jsonb_array_elements_text('["a", "b", "c"]'::jsonb);
-- Returns text rows: a, b, c

-- Expand to key-value pairs
SELECT * FROM jsonb_each('{"a": 1, "b": 2}'::jsonb);
-- Returns: (a, 1), (b, 2)

SELECT * FROM jsonb_each_text('{"a": 1, "b": 2}'::jsonb);
-- Returns text: (a, '1'), (b, '2')
```

## COALESCE for Defaults

```sql
-- Provide default values for missing keys
SELECT 
    COALESCE(data->>'email', 'no-email') AS email,
    COALESCE((data->>'age')::int, 0) AS age,
    COALESCE(data->'settings', '{}'::jsonb) AS settings
FROM users;
```

## Code Examples

```undefined
-- Clean API response before storing
INSERT INTO api_cache (endpoint, response)
VALUES (
    '/users',
    jsonb_strip_nulls('{"users": [{"id": 1, "name": "Alice", "deleted": null}]}'::jsonb)
);

-- Expand JSON array to rows for processing
SELECT 
    o.id AS order_id,
    item->>'product' AS product,
    (item->>'quantity')::int AS quantity
FROM orders o,
    jsonb_array_elements(o.data->'items') AS item
WHERE o.status = 'pending';
```


## Resources

- [JSON Processing Functions](https://www.postgresql.org/docs/current/functions-json.html) — undefined

---

> 📘 *This lesson is part of the [PostgreSQL JSON & Document Processing](https://stanza.dev/courses/postgresql-json) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*