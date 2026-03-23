---
source_course: "postgresql-json"
source_lesson: "postgresql-json-serialize-constructor"
---

# JSON_SERIALIZE, IS JSON, and JSON Constructor

## Introduction
PostgreSQL 16–18 introduced several SQL-standard functions for JSON creation, serialization, and validation. These complement the query functions by providing standardized ways to produce and validate JSON data. Together, they form a complete SQL-standard JSON toolkit.

## Key Concepts
- **JSON_SERIALIZE()**: Converts a JSON-typed value to a text string, with optional formatting. The SQL-standard way to serialize JSON to text.
- **IS JSON predicate**: Tests whether a string value is valid JSON, optionally checking for a specific type (OBJECT, ARRAY, SCALAR) and unique keys.
- **JSON() constructor**: Creates a JSON value from a text string with validation, providing a safe alternative to `::jsonb` casting.
- **WITH UNIQUE KEYS**: An option for `IS JSON` and `JSON()` that additionally verifies no duplicate keys exist.

## Real World Context
When building APIs that accept JSON input from users, you need to validate the input before storing it. The `IS JSON` predicate and `JSON()` constructor provide clean, SQL-standard validation. `JSON_SERIALIZE` is useful when you need to output JSON as text for logging, message queues, or file exports. These functions are increasingly important as more teams adopt SQL-standard syntax for cross-database portability.

## Deep Dive

### JSON_SERIALIZE: JSON to Text

`JSON_SERIALIZE` converts a JSON value to its text representation:

```sql
-- Basic serialization
SELECT JSON_SERIALIZE('{"name": "Alice", "age": 30}'::jsonb);
-- Result: '{"age": 30, "name": "Alice"}' (text)

-- With RETURNING type
SELECT JSON_SERIALIZE('{"a": 1}'::jsonb RETURNING text);
-- Result: '{"a": 1}' (text)

-- Useful for exporting or logging
INSERT INTO audit_log (event_type, payload_text)
SELECT 'user_update', JSON_SERIALIZE(payload)
FROM events
WHERE event_type = 'user_update';
```

While you can cast with `::text`, `JSON_SERIALIZE` is the SQL-standard approach and produces guaranteed valid JSON text.

### IS JSON Predicate: Validate JSON Input

The `IS JSON` predicate tests whether a text string contains valid JSON:

```sql
-- Basic validation
SELECT '{"name": "Alice"}'::text IS JSON;        -- true
SELECT 'not valid json'::text IS JSON;             -- false

-- Check specific types
SELECT '{"a": 1}'::text IS JSON OBJECT;           -- true
SELECT '[1, 2, 3]'::text IS JSON ARRAY;           -- true
SELECT '42'::text IS JSON SCALAR;                  -- true
SELECT '{"a": 1}'::text IS JSON SCALAR;           -- false

-- Unique keys validation
SELECT '{"a": 1, "a": 2}'::text IS JSON WITH UNIQUE KEYS;  -- false
SELECT '{"a": 1, "b": 2}'::text IS JSON WITH UNIQUE KEYS;  -- true
```

Use `IS JSON` in CHECK constraints or WHERE clauses to filter valid JSON rows.

### JSON() Constructor: Safe JSON Creation

The `JSON()` constructor creates a JSON value from text with validation:

```sql
-- Create JSON with validation
SELECT JSON('{"name": "Alice"}');
-- Result: {"name": "Alice"} (json type)

-- Unique keys enforcement
SELECT JSON('{"a": 1, "b": 2}' WITH UNIQUE KEYS);
-- Succeeds

SELECT JSON('{"a": 1, "a": 2}' WITH UNIQUE KEYS);
-- ERROR: duplicate JSON object key value
```

Unlike `::jsonb` casting, the `JSON()` constructor offers the `WITH UNIQUE KEYS` option to reject documents with duplicate keys at creation time.

### Combining These Tools

Here is a pattern for safely accepting, validating, and storing user-provided JSON:

```sql
-- Validate and store user input
INSERT INTO user_settings (user_id, preferences)
SELECT 
    $1,
    CASE 
        WHEN $2::text IS JSON OBJECT 
        THEN $2::jsonb
        ELSE '{}'::jsonb
    END;

-- Export JSON as formatted text for logging
SELECT 
    id,
    JSON_SERIALIZE(settings) AS settings_text
FROM user_settings
WHERE updated_at > now() - interval '1 day';
```

This ensures bad input never enters the database while providing meaningful defaults.

## Common Pitfalls
1. **Confusing IS JSON with IS NOT NULL** — A NULL value is not valid JSON. `NULL IS JSON` returns false (or NULL depending on context), not true.
2. **Expecting JSON() to accept invalid input silently** — Unlike `::jsonb` which also rejects invalid JSON, `JSON()` with `WITH UNIQUE KEYS` adds the additional duplicate key check.
3. **Using JSON_SERIALIZE where ::text works** — For simple `jsonb` to text conversion, `::text` casting is sufficient. Use `JSON_SERIALIZE` when you need standard compliance or specific RETURNING options.

## Best Practices
1. **Use IS JSON in CHECK constraints for text columns** — If a text column must contain valid JSON, add `CHECK (column IS JSON)` for database-level validation.
2. **Use JSON() with UNIQUE KEYS for strict input** — When duplicate keys would cause bugs, enforce uniqueness at construction time.
3. **Prefer JSON_SERIALIZE for cross-database compatibility** — If your SQL needs to run on multiple database engines, use standard functions.

## Summary
- `JSON_SERIALIZE()` converts JSON values to text in a SQL-standard way.
- `IS JSON` validates text as JSON with optional type and unique-key checks.
- `JSON()` constructs JSON from text with optional `WITH UNIQUE KEYS` enforcement.
- These functions complement `JSON_VALUE`, `JSON_QUERY`, and `JSON_TABLE` to form a complete SQL-standard JSON toolkit.
- Available progressively from PostgreSQL 16 (`IS JSON`) through PostgreSQL 17+ (`JSON_SERIALIZE`, `JSON()`).

## Code Examples

**JSON Validation and Serialization**

```sql
-- Validate and filter valid JSON from a text import table
SELECT 
    id,
    raw_input,
    raw_input IS JSON AS is_valid,
    raw_input IS JSON OBJECT AS is_object
FROM imported_data
WHERE raw_input IS JSON;

-- Serialize jsonb to text for message queue
SELECT JSON_SERIALIZE(
    jsonb_build_object(
        'event', 'user_created',
        'user_id', u.id,
        'email', u.email
    )
) AS message_payload
FROM users u
WHERE u.created_at > now() - interval '1 hour';
```


## Resources

- [JSON Functions and Operators](https://www.postgresql.org/docs/current/functions-json.html) — undefined

---

> 📘 *This lesson is part of the [PostgreSQL JSON & Document Processing](https://stanza.dev/courses/postgresql-json) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*