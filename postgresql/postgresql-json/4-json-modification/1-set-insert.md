---
source_course: "postgresql-json"
source_lesson: "jsonb-set-insert"
---

# Updating JSON with jsonb_set and jsonb_insert

JSONB values are immutable - you can't modify them in place. Instead, use functions that return a new modified JSONB value.

## jsonb_set: Update or Add Values

```sql
-- Basic syntax
jsonb_set(target, path, new_value, create_if_missing)

-- Update existing key
SELECT jsonb_set(
    '{"name": "Alice", "age": 30}'::jsonb,
    '{age}',
    '31'
);
-- Result: {"age": 31, "name": "Alice"}

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

## jsonb_insert: Insert Into Arrays

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

## Using in UPDATE Statements

```sql
-- Update a JSON field
UPDATE products
SET attributes = jsonb_set(
    attributes,
    '{stock}',
    to_jsonb(attributes->>'stock'::int - 1)
)
WHERE id = 1;

-- Add new attribute
UPDATE products
SET attributes = jsonb_set(
    attributes,
    '{last_updated}',
    to_jsonb(now())
);
```

## Code Examples

```undefined
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