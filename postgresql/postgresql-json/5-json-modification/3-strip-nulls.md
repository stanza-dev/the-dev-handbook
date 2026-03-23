---
source_course: "postgresql-json"
source_lesson: "postgresql-json-strip-nulls"
---

# Cleaning JSON Data

## Introduction
Real-world JSON data often contains null values, inconsistent formatting, and mixed types that need cleaning before use. PostgreSQL provides functions to strip nulls, format JSON for readability, and convert between JSON and relational formats. PostgreSQL 18 enhances these capabilities with new options for null handling.

## Key Concepts
- **jsonb_strip_nulls()**: Recursively removes all object keys that have null values from a JSONB document.
- **strip_in_arrays parameter (PG18)**: A new boolean parameter in PostgreSQL 18 that extends `jsonb_strip_nulls` to also remove null elements from arrays.
- **jsonb_pretty()**: Formats a JSONB value as indented, human-readable text.
- **jsonb_array_elements() / jsonb_each()**: Set-returning functions that expand arrays to rows or objects to key-value pairs.
- **Improved null-to-scalar casting (PG18)**: PostgreSQL 18 returns SQL NULL instead of an error when casting a JSONB null to a scalar type.

## Real World Context
API responses and imported data frequently contain null fields that inflate storage and complicate queries. Stripping nulls before storage reduces column size and simplifies downstream processing. The PostgreSQL 18 `strip_in_arrays` parameter addresses a long-standing pain point where null array elements survived `jsonb_strip_nulls`. Data pipeline teams use `jsonb_array_elements` and `jsonb_each` to convert JSON into relational form for analytics.

## Deep Dive

### jsonb_strip_nulls: Remove Null Values

This function recursively removes keys with null values from objects:

```sql
-- Remove null keys recursively
SELECT jsonb_strip_nulls(
    '{"a": 1, "b": null, "c": {"d": null, "e": 2}}'::jsonb
);
-- Result: {"a": 1, "c": {"e": 2}}
```

Note that `jsonb_strip_nulls` removes keys with null values from objects but, by default, leaves null elements in arrays untouched.

### PostgreSQL 18: strip_in_arrays Parameter

PostgreSQL 18 adds a second parameter to `jsonb_strip_nulls` that also removes null elements from arrays:

```sql
-- Default behavior: nulls in arrays preserved
SELECT jsonb_strip_nulls(
    '{"items": [1, null, 3, null, 5]}'::jsonb
);
-- Result: {"items": [1, null, 3, null, 5]}

-- PG18: strip_in_arrays removes array nulls too
SELECT jsonb_strip_nulls(
    '{"items": [1, null, 3, null, 5]}'::jsonb,
    true  -- strip_in_arrays
);
-- Result: {"items": [1, 3, 5]}

-- Works recursively on nested structures
SELECT jsonb_strip_nulls(
    '{"data": {"tags": ["a", null, "b"], "meta": null}}'::jsonb,
    true
);
-- Result: {"data": {"tags": ["a", "b"]}}
```

This is particularly useful for cleaning up imported data or API responses that contain sparse arrays.

### PostgreSQL 18: Improved JSONB Null-to-Scalar Casting

PostgreSQL 18 improves how JSONB null values are handled when casting to SQL types:

```sql
-- PG17 and earlier: error when casting jsonb null to scalar
-- SELECT ('null'::jsonb)::text;  -- ERROR

-- PG18: returns SQL NULL instead of error
SELECT ('null'::jsonb)::text;    -- NULL
SELECT ('null'::jsonb)::integer; -- NULL
```

This change eliminates a common source of runtime errors when processing JSON data that may contain null values.

### jsonb_pretty: Format for Readability

Format JSONB as indented text for debugging and logging:

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

This is invaluable for debugging complex JSON structures in psql.

### Type Conversions

Convert between JSON and relational formats:

```sql
-- Convert JSONB array to rows
SELECT * FROM jsonb_array_elements('[1, 2, 3]'::jsonb);
-- Returns rows: 1, 2, 3

SELECT * FROM jsonb_array_elements_text('["a", "b", "c"]'::jsonb);
-- Returns text rows: a, b, c

-- Expand object to key-value pairs
SELECT * FROM jsonb_each('{"a": 1, "b": 2}'::jsonb);
-- Returns: (a, 1), (b, 2)

SELECT * FROM jsonb_each_text('{"a": 1, "b": 2}'::jsonb);
-- Returns text: (a, '1'), (b, '2')
```

These functions are essential for bridging JSON data into relational queries.

### COALESCE for Defaults

Always provide default values for missing keys:

```sql
-- Provide default values for missing keys
SELECT 
    COALESCE(data->>'email', 'no-email') AS email,
    COALESCE((data->>'age')::int, 0) AS age,
    COALESCE(data->'settings', '{}'::jsonb) AS settings
FROM users;
```

This prevents NULL propagation in downstream calculations.

## Common Pitfalls
1. **Expecting jsonb_strip_nulls to remove array nulls without strip_in_arrays** — By default, only object key nulls are removed. Use the PG18 `true` parameter for arrays.
2. **Using jsonb_pretty in production queries** — It adds significant overhead and should only be used for debugging or logging, not in hot paths.
3. **Confusing JSONB null with SQL NULL** — `'{"a": null}'::jsonb->'a'` returns a JSONB null value, not SQL NULL. Use `->>` and COALESCE for safe extraction.

## Best Practices
1. **Strip nulls before storing imported data** — Run `jsonb_strip_nulls` on API responses to reduce storage and simplify queries.
2. **Use strip_in_arrays on PG18+ for thorough cleaning** — When your data contains sparse arrays, enable the new parameter.
3. **Always COALESCE extracted values** — Provide sensible defaults for optional JSON fields to avoid NULL-related bugs.

## Summary
- `jsonb_strip_nulls` removes object keys with null values recursively.
- PostgreSQL 18 adds the `strip_in_arrays` parameter to also remove null array elements.
- PostgreSQL 18 improves null-to-scalar casting to return SQL NULL instead of errors.
- `jsonb_pretty` formats JSON for human readability (debugging only).
- `jsonb_array_elements` and `jsonb_each` convert JSON to relational rows.
- Always use COALESCE when extracting optional JSON values.

## Code Examples

**JSON Cleaning Functions**

```sql
-- Clean API response before storing (PG18 with strip_in_arrays)
INSERT INTO api_cache (endpoint, response)
VALUES (
    '/users',
    jsonb_strip_nulls(
        '{"users": [{"id": 1, "name": "Alice", "deleted": null}],
          "errors": [null, null]}'::jsonb,
        true  -- strip_in_arrays (PG18)
    )
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