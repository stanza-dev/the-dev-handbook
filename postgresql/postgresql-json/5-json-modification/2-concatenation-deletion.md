---
source_course: "postgresql-json"
source_lesson: "postgresql-json-concatenation-deletion"
---

# Concatenation (||) and Deletion (-) Operators

## Introduction
Beyond `jsonb_set`, PostgreSQL provides operators for merging and trimming JSONB values. The concatenation operator (`||`) merges objects or appends arrays, while the deletion operator (`-`) removes keys or elements. These operators are concise and expressive, making common modifications a single operation.

## Key Concepts
- **Concatenation (`||`)**: Merges two JSONB values. For objects, right-side keys overwrite left-side keys. For arrays, elements are appended.
- **Key deletion (`-`)**: Removes a key from a JSONB object by name, or an element from an array by index.
- **Multi-key deletion (`- array[...]`)**: Removes multiple keys at once from a JSONB object.
- **Path deletion (`#-`)**: Removes a value at a specified nested path.

## Real World Context
Merging user-provided settings with system defaults is a classic use case for `||`. When a user updates their preferences, you merge the new values over the defaults. The deletion operator is essential for data sanitization — removing sensitive fields like passwords or tokens before returning data to clients.

## Deep Dive

### Concatenation Operator (||)

The `||` operator merges JSONB values with right-side priority:

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

For objects, `||` performs a shallow merge. Nested objects are replaced entirely, not merged recursively.

### Deletion Operators

The `-` operator removes keys or elements:

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

The path deletion operator (`#-`) is essential for removing nested fields without affecting the rest of the document.

### Practical Examples

Common patterns in production:

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

Note that in the preferences merge, the default values are on the left, so user preferences on the right take priority.

## Common Pitfalls
1. **Expecting deep merge with ||** — The `||` operator does a shallow merge. `'{"a": {"b": 1}}' || '{"a": {"c": 2}}'` replaces the entire "a" object, losing "b".
2. **Confusing integer deletion with key deletion** — `'{"1": "a"}' - 1` removes the key "1" (by integer index), which behaves differently from `- '1'` (by key name).
3. **Using - with an index on an object** — Integer deletion on objects removes by position, not by key value, which is rarely what you want.

## Best Practices
1. **Use || for merging settings with defaults** — Place defaults on the left and user values on the right for correct override behavior.
2. **Use - array[...] for sanitizing output** — Remove multiple sensitive keys in one operation before returning data.
3. **Use #- for nested deletions** — It is the only operator that can remove a value at a specific nested path.

## Summary
- `||` merges JSONB values with right-side priority for conflicting keys.
- `-` removes keys by name, multiple keys by array, or elements by index.
- `#-` removes values at nested paths.
- `||` performs shallow merge only; nested objects are replaced, not recursively merged.
- Use these operators for settings merge, data sanitization, and structural modifications.

## Code Examples

**Concatenation and Deletion Examples**

```sql
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