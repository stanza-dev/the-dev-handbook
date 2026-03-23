---
source_course: "postgresql-json"
source_lesson: "postgresql-json-json-validation"
---

# JSON Validation and Type Checking

## Introduction
Storing flexible JSON data is powerful, but without validation you risk inconsistent documents that break application logic. PostgreSQL provides several tools to validate JSON structure and types, from simple type-checking functions to the SQL/JSON `IS JSON` predicate introduced in recent versions.

## Key Concepts
- **jsonb_typeof()**: A function that returns the top-level type of a JSONB value as a string (object, array, string, number, boolean, or null).
- **IS JSON predicate**: A SQL-standard predicate (PostgreSQL 16+) that checks whether a string is valid JSON, optionally verifying it is a specific type.
- **CHECK constraints**: Table constraints that enforce structural rules on JSONB columns at the database level.
- **Schema validation**: The practice of ensuring JSON documents conform to an expected structure before or during storage.

## Real World Context
APIs receive data from external clients that may send malformed or unexpected JSON. By adding validation at the database level, you create a safety net that prevents bad data from entering your system regardless of which application writes to the table. Healthcare and financial applications commonly enforce strict JSON schemas via CHECK constraints.

## Deep Dive

### jsonb_typeof for Runtime Checks

The `jsonb_typeof` function returns the type of the top-level JSON value as text:

```sql
SELECT jsonb_typeof('{"a": 1}'::jsonb);    -- 'object'
SELECT jsonb_typeof('[1, 2, 3]'::jsonb);    -- 'array'
SELECT jsonb_typeof('"hello"'::jsonb);      -- 'string'
SELECT jsonb_typeof('42'::jsonb);            -- 'number'
SELECT jsonb_typeof('true'::jsonb);          -- 'boolean'
SELECT jsonb_typeof('null'::jsonb);          -- 'null'
```

This is useful for conditional logic in queries and for enforcing structure.

### The IS JSON Predicate (PostgreSQL 16+)

The `IS JSON` predicate tests whether a text value is valid JSON:

```sql
-- Check if a string is valid JSON
SELECT '{"name": "Alice"}'::text IS JSON;            -- true
SELECT 'not json'::text IS JSON;                       -- false

-- Check for specific JSON types
SELECT '{"a": 1}'::text IS JSON OBJECT;               -- true
SELECT '[1, 2]'::text IS JSON ARRAY;                   -- true
SELECT '42'::text IS JSON SCALAR;                      -- true

-- With UNIQUE KEYS check
SELECT '{"a": 1, "a": 2}'::text IS JSON WITH UNIQUE KEYS;  -- false
```

This predicate is particularly useful for validating text input before casting to jsonb.

### CHECK Constraints for Structure Enforcement

Combine `jsonb_typeof` with CHECK constraints to enforce document structure at the table level:

```sql
CREATE TABLE configurations (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    settings JSONB NOT NULL,
    
    -- Ensure settings is always an object
    CONSTRAINT settings_is_object 
        CHECK (jsonb_typeof(settings) = 'object'),
    
    -- Ensure required keys exist
    CONSTRAINT settings_has_required_keys
        CHECK (settings ? 'version' AND settings ? 'enabled')
);

-- This succeeds
INSERT INTO configurations (name, settings) 
VALUES ('app', '{"version": "1.0", "enabled": true}');

-- This fails: missing 'version' key
INSERT INTO configurations (name, settings) 
VALUES ('app', '{"enabled": true}');
```

These constraints run on every INSERT and UPDATE, guaranteeing data consistency.

### Validating Nested Structure

For deeper validation, combine multiple checks:

```sql
CREATE TABLE api_events (
    id SERIAL PRIMARY KEY,
    payload JSONB NOT NULL,
    
    CONSTRAINT payload_is_object
        CHECK (jsonb_typeof(payload) = 'object'),
    CONSTRAINT payload_has_type
        CHECK (payload ? 'type' AND jsonb_typeof(payload->'type') = 'string'),
    CONSTRAINT payload_has_timestamp
        CHECK (payload ? 'timestamp')
);
```

This ensures every event has the minimum required structure.

## Common Pitfalls
1. **Relying only on application-level validation** — Applications have bugs and multiple services may write to the same table. Database constraints are the last line of defense.
2. **Confusing jsonb_typeof('null') with SQL NULL** — A JSONB `null` is a valid value with type 'null', but it is not the same as a SQL NULL column value.
3. **Over-constraining JSONB columns** — Adding too many CHECK constraints defeats the purpose of using flexible JSON storage. Validate required structure, not every optional field.

## Best Practices
1. **Always add a type CHECK** — At minimum, ensure your JSONB column contains the expected top-level type (object vs array).
2. **Validate required keys with the ? operator** — Use `CHECK (column ? 'key')` for fields that must always be present.
3. **Use IS JSON for text input validation** — Before casting user-provided text to jsonb, validate with `IS JSON` to produce clear error messages.

## Summary
- `jsonb_typeof()` returns the top-level type of a JSONB value as text.
- The `IS JSON` predicate (PostgreSQL 16+) validates text strings as JSON with optional type and uniqueness checks.
- CHECK constraints enforce JSONB structure at the database level.
- Combine type checks and key existence checks for robust validation.
- Database-level validation complements application-level checks as a safety net.

## Code Examples

**JSON Validation Patterns**

```sql
-- Validate text input before casting
SELECT 
    input_text,
    input_text IS JSON AS is_valid,
    CASE 
        WHEN input_text IS JSON OBJECT THEN 'object'
        WHEN input_text IS JSON ARRAY THEN 'array'
        WHEN input_text IS JSON SCALAR THEN 'scalar'
        ELSE 'invalid'
    END AS json_type
FROM (VALUES 
    ('{"a": 1}'),
    ('[1, 2]'),
    ('42'),
    ('not json')
) AS t(input_text);
```


## Resources

- [JSON Functions and Operators](https://www.postgresql.org/docs/current/functions-json.html) — undefined

---

> 📘 *This lesson is part of the [PostgreSQL JSON & Document Processing](https://stanza.dev/courses/postgresql-json) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*