---
source_course: "postgresql-json"
source_lesson: "json-concatenation-deletion"
---

# Concatenation and Deletion Operators

JSONB supports operators for combining values and removing keys.

## Concatenation Operator (||)

```sql
-- Merge objects (right overwrites left)
SELECT '{"a": 1}'::jsonb || '{"b": 2}'::jsonb;
-- Result: {"a": 1, "b": 2}

SELECT '{"a": 1}'::jsonb || '{"a": 2}'::jsonb;
-- Result: {"a": 2}

-- Concatenate arrays
SELECT '[1, 2]'::jsonb || '[3, 4]'::jsonb;
-- Result: [1, 2, 3, 4]

-- Add element to array
SELECT '[1, 2]'::jsonb || '3'::jsonb;
-- Result: [1, 2, 3]
```

## Deletion Operators

```sql
-- Remove key from object
SELECT '{"a": 1, "b": 2, "c": 3}'::jsonb - 'b';
-- Result: {"a": 1, "c": 3}

-- Remove multiple keys
SELECT '{"a": 1, "b": 2, "c": 3}'::jsonb - array['a', 'c'];
-- Result: {"b": 2}

-- Remove element by index from array
SELECT '[0, 1, 2, 3]'::jsonb - 1;
-- Result: [0, 2, 3]

-- Remove by path
SELECT '{"a": {"b": 1, "c": 2}}'::jsonb #- '{a,b}';
-- Result: {"a": {"c": 2}}
```

## Practical Examples

```sql
-- Merge user preferences with defaults
UPDATE users
SET preferences = '{"theme": "dark", "notifications": true}'::jsonb || preferences;

-- Remove sensitive data before returning
SELECT data - array['password', 'ssn', 'credit_card'] AS safe_data
FROM users;

-- Remove nested field
UPDATE profiles
SET data = data #- '{address,zip_code}';
```

## Code Examples

```undefined
-- Merge settings with defaults
SELECT 
    '{"theme": "light", "fontSize": 14}'::jsonb || 
    '{"theme": "dark", "language": "en"}'::jsonb AS merged;
-- {"theme": "dark", "fontSize": 14, "language": "en"}

-- Remove multiple sensitive fields
SELECT 
    user_data - array['password_hash', 'secret_key', 'api_token']
FROM users;

-- Clean up nested structure
UPDATE documents
SET metadata = metadata #- '{internal,debug_info}'
WHERE metadata ? 'internal';
```


## Resources

- [JSONB Operators](https://www.postgresql.org/docs/current/functions-json.html#FUNCTIONS-JSON-OP-TABLE) — undefined

---

> 📘 *This lesson is part of the [PostgreSQL JSON & Document Processing](https://stanza.dev/courses/postgresql-json) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*