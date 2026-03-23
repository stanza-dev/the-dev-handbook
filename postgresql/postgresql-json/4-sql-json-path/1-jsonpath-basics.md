---
source_course: "postgresql-json"
source_lesson: "postgresql-json-jsonpath-basics"
---

# JSONPath Fundamentals

## Introduction
SQL/JSON path is a powerful query language for navigating and filtering JSON data, standardized in SQL:2016. It provides a concise syntax for selecting elements, filtering by conditions, and transforming results — all within a single path expression. This lesson covers the core syntax that underpins all JSONPath operations.

## Key Concepts
- **`$` (root)**: The symbol representing the root of the JSON document. Every path expression starts with `$`.
- **Dot notation (`$.key`)**: Accesses an object member by key name.
- **Bracket notation (`$[0]`)**: Accesses array elements by index, with `[*]` for all elements.
- **Lax vs Strict mode**: Lax mode (default) silently handles missing keys; strict mode raises errors.
- **Recursive descent (`$..key`)**: Finds all occurrences of a key at any depth in the document.

## Real World Context
JSONPath is used across the industry for querying JSON data — not just in PostgreSQL but in REST APIs (JSONPath is a standard query format), configuration tools, and log analysis. Learning PostgreSQL's implementation gives you transferable skills. Complex queries that would require multiple subqueries with extraction operators can often be expressed as a single JSONPath expression.

## Deep Dive

### Path Expression Basics

Every JSONPath expression starts with `$` to reference the root document:

```sql
-- $ represents the root object
SELECT jsonb_path_query('{"name": "Alice", "age": 30}'::jsonb, '$.name');
-- Result: "Alice"

-- .key accesses object members
SELECT jsonb_path_query(
    '{"user": {"name": "Alice", "email": "alice@example.com"}}'::jsonb,
    '$.user.email'
);
-- Result: "alice@example.com"
```

Dot notation is the most common way to navigate into nested objects.

### Array Access

Bracket notation handles arrays:

```sql
-- [*] accesses all array elements
SELECT jsonb_path_query_array(
    '[{"name": "Alice"}, {"name": "Bob"}]'::jsonb,
    '$[*].name'
);
-- Result: ["Alice", "Bob"]

-- [n] accesses specific array index
SELECT jsonb_path_query('[1, 2, 3, 4, 5]'::jsonb, '$[2]');
-- Result: 3
```

The `[*]` wildcard iterates over all array elements, producing one result per element.

### JSONPath Functions Overview

PostgreSQL provides five functions that use JSONPath:

| Function | Returns | Use Case |
|----------|---------|----------|
| `jsonb_path_query()` | SETOF jsonb | Multiple results as rows |
| `jsonb_path_query_array()` | jsonb array | All results in one array |
| `jsonb_path_query_first()` | jsonb | First match only |
| `jsonb_path_exists()` | boolean | Check if path matches |
| `jsonb_path_match()` | boolean | Check if predicate is true |

Choose the function based on what you need: rows, an array, a boolean, or just the first match.

### Path Modes: Lax vs Strict

The mode controls how missing keys are handled:

```sql
-- Lax mode (default): tolerates missing keys
SELECT jsonb_path_query('{"name": "Alice"}'::jsonb, 'lax $.email');
-- Returns: (no rows)

-- Strict mode: error on missing keys
SELECT jsonb_path_query('{"name": "Alice"}'::jsonb, 'strict $.email');
-- ERROR: JSON object does not contain key "email"
```

Lax mode is the default and is appropriate for most queries. Use strict mode when you want to enforce that paths exist.

### Recursive Descent with `..`

The `..` operator finds keys at any depth:

```sql
-- Find all 'name' keys at any depth
SELECT jsonb_path_query(
    '{
        "company": "Acme",
        "employees": [
            {"name": "Alice", "manager": {"name": "Bob"}},
            {"name": "Charlie"}
        ]
    }'::jsonb,
    '$..name'
);
-- Results: "Acme", "Alice", "Bob", "Charlie"
```

Recursive descent is powerful but should be used carefully — it traverses the entire document.

## Common Pitfalls
1. **Omitting $ at the start** — Every JSONPath expression must begin with `$`. Writing `name` instead of `$.name` is a syntax error.
2. **Assuming strict mode is the default** — Lax mode is the default, meaning missing keys silently return nothing rather than raising errors.
3. **Using [*] on an object instead of an array** — In lax mode, `$[*]` on an object wraps it as a single-element array and returns the object. This can produce unexpected results.

## Best Practices
1. **Start with lax mode** — It is the safe default for most queries and gracefully handles optional fields.
2. **Use strict mode for validation** — When you need to ensure a path exists, switch to strict mode for explicit error reporting.
3. **Prefer `.key` over `["key"]`** — Dot notation is more readable; use bracket notation only for keys with special characters.

## Summary
- JSONPath expressions start with `$` for the root and use dot notation for object access.
- Array elements are accessed with `[n]` for specific indices or `[*]` for all elements.
- Five PostgreSQL functions accept JSONPath: `jsonb_path_query`, `jsonb_path_query_array`, `jsonb_path_query_first`, `jsonb_path_exists`, and `jsonb_path_match`.
- Lax mode (default) silently handles missing keys; strict mode raises errors.
- Recursive descent (`$..key`) finds matching keys at any depth.

## Code Examples

**JSONPath navigation with dot notation, array access, and recursive descent**

```sql
-- Dot notation for nested access
SELECT jsonb_path_query(
    '{"user": {"name": "Alice", "email": "alice@example.com"}}'::jsonb,
    '$.user.email'
);
-- Result: "alice@example.com"

-- Array access with [*] wildcard
SELECT jsonb_path_query_array(
    '[{"name": "Alice"}, {"name": "Bob"}]'::jsonb,
    '$[*].name'
);
-- Result: ["Alice", "Bob"]

-- Recursive descent finds keys at any depth
SELECT jsonb_path_query(
    '{"team": {"lead": {"name": "Alice"}, "members": [{"name": "Bob"}]}}'::jsonb,
    '$..name'
);
-- Results: "Alice", "Bob"
```


## Resources

- [SQL/JSON Path Language](https://www.postgresql.org/docs/18/functions-json.html#FUNCTIONS-SQLJSON-PATH) — undefined

---

> 📘 *This lesson is part of the [PostgreSQL JSON & Document Processing](https://stanza.dev/courses/postgresql-json) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*