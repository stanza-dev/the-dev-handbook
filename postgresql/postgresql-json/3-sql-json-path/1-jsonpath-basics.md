---
source_course: "postgresql-json"
source_lesson: "jsonpath-basics"
---

# SQL/JSON Path Language

SQL/JSON path is a powerful query language for navigating and filtering JSON data, standardized in SQL:2016.

## Path Expression Basics

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

## JSONPath Functions Overview

| Function | Returns | Use Case |
|----------|---------|----------|
| `jsonb_path_query()` | SETOF jsonb | Multiple results as rows |
| `jsonb_path_query_array()` | jsonb array | All results in one array |
| `jsonb_path_query_first()` | jsonb | First match only |
| `jsonb_path_exists()` | boolean | Check if path matches |
| `jsonb_path_match()` | boolean | Check if predicate is true |

## Path Modes: Lax vs Strict

```sql
-- Lax mode (default): tolerates missing keys
SELECT jsonb_path_query('{"name": "Alice"}'::jsonb, 'lax $.email');
-- Returns: (no rows)

-- Strict mode: error on missing keys
SELECT jsonb_path_query('{"name": "Alice"}'::jsonb, 'strict $.email');
-- ERROR: JSON object does not contain key "email"
```

## Recursive Descent with **

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

📖 [SQL/JSON Path](https://www.postgresql.org/docs/18/functions-json.html#FUNCTIONS-SQLJSON-PATH)

## Resources

- [SQL/JSON Path Language](https://www.postgresql.org/docs/18/functions-json.html#FUNCTIONS-SQLJSON-PATH) — undefined

---

> 📘 *This lesson is part of the [PostgreSQL JSON & Document Processing](https://stanza.dev/courses/postgresql-json) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*