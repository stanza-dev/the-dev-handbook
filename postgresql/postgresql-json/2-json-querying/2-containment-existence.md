---
source_course: "postgresql-json"
source_lesson: "containment-existence"
---

# Containment and Existence Operators

JSONB supports powerful containment and existence operators that are essential for efficient querying.

## Containment Operators

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

## Existence Operators

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

## Practical Queries

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

**Important:** Containment operators work with GIN indexes, making them very fast for large datasets!

## Code Examples

```undefined
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