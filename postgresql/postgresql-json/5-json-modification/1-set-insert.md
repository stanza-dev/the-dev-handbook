---
source_course: "postgresql-json"
source_lesson: "postgresql-json-jsonb-set-insert"
---

# Updating JSON with jsonb_set and jsonb_insert

## Introduction
JSONB values in PostgreSQL are immutable — you cannot modify them in place. Instead, functions like `jsonb_set` and `jsonb_insert` return a new JSONB value with the requested modifications. These functions are the primary tools for updating specific parts of a JSON document without rewriting the entire value.

## Key Concepts
- **jsonb_set()**: Replaces the value at a given path in a JSONB document, or optionally creates a new key if it does not exist.
- **jsonb_insert()**: Inserts a value into a JSONB array at a specified position, either before or after the given index.
- **Immutability**: JSONB values cannot be modified in place. Every modification function returns a new JSONB value.
- **Path notation**: Both functions use `'{key1,key2}'` text array syntax to specify the location of the modification.

## Real World Context
Applications frequently need to update individual fields within a JSON document — incrementing a counter, adding a tag, or updating a nested setting. Without `jsonb_set`, you would need to extract the entire document, modify it in application code, and write it back. This creates race conditions in concurrent environments. Using `jsonb_set` in an UPDATE statement is atomic and avoids these issues.

## Deep Dive

### jsonb_set: Update or Add Values

The function signature is `jsonb_set(target, path, new_value, create_if_missing)`:

```sql
-- Update existing key
SELECT jsonb_set(
    '{"name": "Alice", "age": 30}'::jsonb,
    '{age}',
    '31'
);
-- Result: {"age": 31, "name": "Alice"}
```

The path `'{age}'` targets the top-level "age" key. The value `'31'` is a JSONB literal.

```sql
-- Add new key
SELECT jsonb_set(
    '{"name": "Alice"}'::jsonb,
    '{email}',
    '"alice@example.com"'
);
-- Result: {"name": "Alice", "email": "alice@example.com"}

-- Nested update
SELECT jsonb_set(
    '{"user": {"name": "Alice", "age": 30}}'::jsonb,
    '{user,age}',
    '31'
);
```

For nested paths, list each key in the path array: `'{user,age}'` navigates to `user.age`.

### jsonb_insert: Insert Into Arrays

`jsonb_insert` is designed for array manipulation:

```sql
-- Insert at position (before by default)
SELECT jsonb_insert(
    '{"tags": ["a", "c"]}'::jsonb,
    '{tags,1}',
    '"b"'
);
-- Result: {"tags": ["a", "b", "c"]}

-- Insert after position
SELECT jsonb_insert(
    '{"tags": ["a", "c"]}'::jsonb,
    '{tags,1}',
    '"b"',
    true  -- insert_after
);
-- Result: {"tags": ["a", "c", "b"]}
```

The fourth parameter controls whether insertion is before (default) or after the specified index.

### Using in UPDATE Statements

These functions are most powerful in UPDATE statements where they enable atomic modifications:

```sql
-- Update a specific JSON field atomically
UPDATE products
SET attributes = jsonb_set(
    attributes,
    '{stock}',
    to_jsonb(COALESCE((attributes->>'stock')::int, 0) - 1)
)
WHERE id = 1;

-- Add a timestamp to any document
UPDATE products
SET attributes = jsonb_set(
    attributes,
    '{last_updated}',
    to_jsonb(now()::text)
);
```

Because this runs as a single UPDATE statement, it is safe for concurrent access.

## Common Pitfalls
1. **Forgetting that jsonb_set returns a new value** — You must assign the result back to the column with SET. `jsonb_set(column, ...)` alone does nothing.
2. **Creating intermediate paths that do not exist** — `jsonb_set` cannot create multiple levels at once. If the parent path does not exist, the operation silently does nothing.
3. **Confusing jsonb_set with jsonb_insert for arrays** — `jsonb_set` replaces the value at an array index; `jsonb_insert` adds a new element without removing existing ones.

## Best Practices
1. **Use jsonb_set for field updates** — It is the standard tool for modifying specific keys in a JSONB document.
2. **Use jsonb_insert for array additions** — When adding elements to arrays, `jsonb_insert` preserves existing elements.
3. **Combine with COALESCE for safe defaults** — Always handle the case where the target key might not exist: `COALESCE((data->>'count')::int, 0)`.

## Summary
- `jsonb_set` replaces or adds a value at a specific path in a JSONB document.
- `jsonb_insert` inserts elements into JSONB arrays at a given position.
- Both return new JSONB values (immutability) and must be assigned back to the column.
- Use `jsonb_set` in UPDATE statements for atomic, concurrent-safe modifications.
- The fourth parameter of `jsonb_set` controls whether missing keys are created.

## Code Examples

**JSONB Modification Patterns**

```sql
-- Increment a counter in JSON
UPDATE stats
SET data = jsonb_set(
    data,
    '{views}',
    to_jsonb(COALESCE((data->>'views')::int, 0) + 1)
)
WHERE id = 1;

-- Add item to array
UPDATE users
SET profile = jsonb_set(
    profile,
    '{roles}',
    profile->'roles' || '"admin"'::jsonb
)
WHERE id = 1;

-- Update nested value
UPDATE products
SET attributes = jsonb_set(
    attributes,
    '{specs,ram}',
    '32'
)
WHERE attributes @> '{"brand": "Dell"}';
```


## Resources

- [JSON Functions](https://www.postgresql.org/docs/current/functions-json.html) — undefined

---

> 📘 *This lesson is part of the [PostgreSQL JSON & Document Processing](https://stanza.dev/courses/postgresql-json) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*