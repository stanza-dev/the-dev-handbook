---
source_course: "postgresql-json"
source_lesson: "postgresql-json-containment-existence"
---

# Containment and Existence Operators

## Introduction
Beyond simple extraction, JSONB supports containment and existence operators that test whether a document contains a specific structure or keys. These operators are uniquely powerful because they work with GIN indexes, making them the fastest way to filter JSONB documents in large tables.

## Key Concepts
- **Containment (`@>`)**: Tests whether the left JSONB value contains the right JSONB value as a subset, including nested structures.
- **Contained by (`<@`)**: The reverse of containment — tests whether the left value is contained within the right value.
- **Key existence (`?`)**: Tests whether a specific key (or string element in an array) exists in the top level of a JSONB object.
- **Any key exists (`?|`)**: Tests whether any key in the given array exists in the JSONB value.
- **All keys exist (`?&`)**: Tests whether all keys in the given array exist in the JSONB value.

## Real World Context
Containment queries are the backbone of document-style filtering in PostgreSQL. When an e-commerce site lets users filter products by brand, color, and specs, the backend often uses `@>` with a GIN index for sub-millisecond lookups across millions of products. Key existence operators are used to find documents with optional fields, such as checking which users have set up two-factor authentication.

## Deep Dive

### Containment Operators

The `@>` operator checks whether the left value contains the right value:

```sql
-- @> : Contains (left contains right)
SELECT '{"a": 1, "b": 2}'::jsonb @> '{"a": 1}'::jsonb;  -- true
SELECT '{"a": 1}'::jsonb @> '{"a": 1, "b": 2}'::jsonb;  -- false

-- <@ : Is contained by
SELECT '{"a": 1}'::jsonb <@ '{"a": 1, "b": 2}'::jsonb;  -- true

-- Array containment
SELECT '[1, 2, 3]'::jsonb @> '[1, 3]'::jsonb;  -- true
SELECT '[1, 2, 3]'::jsonb @> '[1, 4]'::jsonb;  -- false
```

Containment checks nested structures recursively, so you can match deeply nested values in a single operator.

### Existence Operators

Existence operators check for the presence of keys:

```sql
-- ? : Key exists
SELECT '{"a": 1, "b": 2}'::jsonb ? 'a';  -- true
SELECT '{"a": 1, "b": 2}'::jsonb ? 'c';  -- false

-- ?| : Any key exists
SELECT '{"a": 1, "b": 2}'::jsonb ?| array['c', 'b'];  -- true

-- ?& : All keys exist
SELECT '{"a": 1, "b": 2}'::jsonb ?& array['a', 'b'];  -- true
SELECT '{"a": 1, "b": 2}'::jsonb ?& array['a', 'c'];  -- false
```

These operators only check top-level keys. To check nested keys, extract the nested object first.

### Practical Queries

Here are common patterns you will use in production:

```sql
-- Find products with specific brand
SELECT * FROM products
WHERE attributes @> '{"brand": "Apple"}';

-- Find users with admin role
SELECT * FROM users
WHERE profile->'roles' @> '["admin"]'::jsonb;

-- Find documents with required fields
SELECT * FROM documents
WHERE metadata ?& array['author', 'created_at', 'version'];

-- Find any matching tag
SELECT * FROM articles
WHERE tags ?| array['postgresql', 'database', 'sql'];
```

All of these queries can use GIN indexes, making them very fast even on large datasets.

## Common Pitfalls
1. **Expecting containment to check nested keys directly** — `'{"a": {"b": 1}}' @> '{"b": 1}'` is false because `@>` checks at the same nesting level. You need `'{"a": {"b": 1}}'::jsonb @> '{"a": {"b": 1}}'::jsonb`.
2. **Using ? with parameterized queries in some ORMs** — The `?` operator conflicts with parameter placeholders in many ORMs and drivers. Use the function form `jsonb_exists(column, key)` or escape the operator.
3. **Forgetting that existence operators check top-level only** — `'{"a": {"b": 1}}'::jsonb ? 'b'` is false because `b` is not a top-level key.

## Best Practices
1. **Use @> with GIN indexes for document filtering** — Containment queries are the most index-friendly JSONB operations and should be your default filtering pattern.
2. **Prefer ?& for required-fields validation** — Rather than multiple `?` checks, use `?& array['key1', 'key2']` for cleaner and slightly faster queries.
3. **Create GIN indexes on columns used with containment** — Without a GIN index, containment operators still do a sequential scan.

## Summary
- `@>` tests whether a JSONB value contains another as a subset (works recursively on nested structures).
- `?`, `?|`, and `?&` test for key existence at the top level.
- All containment and existence operators are supported by GIN indexes.
- Use `@>` as your primary filtering pattern for JSONB document queries.
- Existence operators only check top-level keys, not nested ones.

## Code Examples

**Containment Query Patterns**

```sql
-- Find products matching multiple criteria
SELECT * FROM products
WHERE attributes @> '{
    "brand": "Dell",
    "specs": {"ram": 16}
}'::jsonb;

-- Find users with specific permissions
SELECT * FROM users
WHERE permissions @> '["read", "write"]'::jsonb;

-- Check for optional fields
SELECT 
    id,
    data->>'name' AS name,
    CASE WHEN data ? 'email' THEN data->>'email' ELSE 'N/A' END AS email
FROM profiles;
```


## Resources

- [JSON Containment](https://www.postgresql.org/docs/current/datatype-json.html#JSON-CONTAINMENT) — undefined

---

> 📘 *This lesson is part of the [PostgreSQL JSON & Document Processing](https://stanza.dev/courses/postgresql-json) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*