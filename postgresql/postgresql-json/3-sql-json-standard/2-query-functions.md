---
source_course: "postgresql-json"
source_lesson: "postgresql-json-query-functions"
---

# JSON_VALUE, JSON_QUERY, and JSON_EXISTS

## Introduction
PostgreSQL 17 introduced three SQL-standard query functions that provide a cleaner, more portable alternative to the PostgreSQL-specific `->`, `->>`, and `jsonb_path_exists` patterns. These functions offer built-in type casting, error handling, and consistent behavior across SQL-standard databases.

## Key Concepts
- **JSON_VALUE()**: Extracts a scalar value from a JSON document and returns it as a specified SQL type. Returns NULL or a default for non-scalar results.
- **JSON_QUERY()**: Extracts a JSON object or array from a document, returning the result as jsonb. Designed for non-scalar extraction.
- **JSON_EXISTS()**: Tests whether a JSON path expression matches any items, returning a boolean. Equivalent to `@?` but with error handling options.
- **RETURNING clause**: Specifies the output SQL type for `JSON_VALUE` and `JSON_QUERY`, enabling direct type casting.

## Real World Context
These functions are part of the SQL/JSON standard (SQL:2016), which means queries written with them are portable across PostgreSQL, Oracle, MySQL 8+, and SQL Server. If your organization uses multiple database engines or plans to migrate, SQL-standard functions reduce rewrite effort. They also simplify type casting that previously required manual `::type` operators.

## Deep Dive

### JSON_VALUE: Extract Scalars with Type Safety

`JSON_VALUE` extracts a scalar (string, number, boolean, null) and casts it to a SQL type:

```sql
-- Basic scalar extraction
SELECT JSON_VALUE('{"name": "Alice", "age": 30}'::jsonb, '$.name');
-- Result: 'Alice' (text)

-- With explicit RETURNING type
SELECT JSON_VALUE('{"price": 29.99}'::jsonb, '$.price' RETURNING numeric);
-- Result: 29.99 (numeric)

-- With default on error
SELECT JSON_VALUE(
    '{"data": "not_a_number"}'::jsonb,
    '$.data' RETURNING integer DEFAULT 0 ON ERROR
);
-- Result: 0

-- NULL ON EMPTY for missing paths
SELECT JSON_VALUE('{"a": 1}'::jsonb, '$.missing' RETURNING text NULL ON EMPTY);
-- Result: NULL
```

`JSON_VALUE` is designed for scalar extraction. If the path points to an object or array, it returns NULL (or the default on error).

### JSON_QUERY: Extract Objects and Arrays

`JSON_QUERY` extracts non-scalar values (objects, arrays) and returns them as jsonb:

```sql
-- Extract a nested object
SELECT JSON_QUERY(
    '{"user": {"name": "Alice", "email": "alice@example.com"}}'::jsonb,
    '$.user'
);
-- Result: {"name": "Alice", "email": "alice@example.com"}

-- Extract an array
SELECT JSON_QUERY('{"tags": ["sql", "postgresql", "json"]}'::jsonb, '$.tags');
-- Result: ["sql", "postgresql", "json"]

-- With WRAPPER for collecting multiple matches
SELECT JSON_QUERY(
    '[{"id": 1}, {"id": 2}, {"id": 3}]'::jsonb,
    '$[*].id' WITH ARRAY WRAPPER
);
-- Result: [1, 2, 3]
```

Use `JSON_QUERY` when you need to preserve the JSON structure of the extracted data.

### JSON_EXISTS: Path Existence Check

`JSON_EXISTS` returns a boolean indicating whether the path matches:

```sql
-- Check if a path exists
SELECT JSON_EXISTS('{"a": {"b": 1}}'::jsonb, '$.a.b');
-- Result: true

-- With filter
SELECT JSON_EXISTS(
    '{"items": [{"price": 10}, {"price": 50}]}'::jsonb,
    '$.items[*] ? (@.price > 30)'
);
-- Result: true

-- Use in WHERE clause
SELECT * FROM products
WHERE JSON_EXISTS(attributes, '$.specs.ram ? (@ >= 16)');
```

`JSON_EXISTS` is the SQL-standard equivalent of `jsonb_path_exists()` and the `@?` operator.

### Comparison with PostgreSQL-Specific Syntax

| SQL/JSON Standard | PostgreSQL-Specific | Notes |
|---|---|---|
| `JSON_VALUE(col, '$.name')` | `col->>'name'` | Standard has RETURNING and error handling |
| `JSON_QUERY(col, '$.addr')` | `col->'addr'` | Standard has WRAPPER options |
| `JSON_EXISTS(col, '$.key')` | `col ? 'key'` | Standard has error handling |

Both forms work in PostgreSQL. The standard functions offer more features; the operators are more concise.

## Common Pitfalls
1. **Using JSON_VALUE for objects or arrays** — `JSON_VALUE` only extracts scalars. For nested objects or arrays, use `JSON_QUERY`.
2. **Forgetting RETURNING for numeric extraction** — Without `RETURNING numeric`, `JSON_VALUE` returns text, which then requires a separate cast.
3. **Expecting these functions in PostgreSQL 16 or earlier** — `JSON_VALUE`, `JSON_QUERY`, and `JSON_EXISTS` require PostgreSQL 17 or later.

## Best Practices
1. **Use JSON_VALUE with RETURNING for type-safe extraction** — It combines extraction and casting in one step, reducing errors.
2. **Choose the right function** — `JSON_VALUE` for scalars, `JSON_QUERY` for objects/arrays, `JSON_EXISTS` for boolean checks.
3. **Add error handling for untrusted data** — Use `DEFAULT ... ON ERROR` and `NULL ON EMPTY` to gracefully handle unexpected structures.

## Summary
- `JSON_VALUE` extracts scalar values with built-in type casting via RETURNING.
- `JSON_QUERY` extracts objects and arrays, preserving JSON structure.
- `JSON_EXISTS` checks whether a JSON path matches, returning a boolean.
- All three support error handling clauses (DEFAULT ON ERROR, NULL ON EMPTY).
- These are SQL-standard functions available in PostgreSQL 17+, portable across databases.

## Code Examples

**SQL/JSON Query Functions Comparison**

```sql
-- JSON_VALUE for typed scalar extraction
SELECT 
    JSON_VALUE(data, '$.customer_name') AS name,
    JSON_VALUE(data, '$.total' RETURNING numeric) AS total,
    JSON_VALUE(data, '$.order_date' RETURNING date) AS order_date
FROM orders;

-- JSON_QUERY for structure extraction
SELECT 
    id,
    JSON_QUERY(data, '$.shipping_address') AS address
FROM orders
WHERE JSON_EXISTS(data, '$.shipping_address.zip');
```


## Resources

- [SQL/JSON Query Functions](https://www.postgresql.org/docs/current/functions-json.html#FUNCTIONS-SQLJSON-QUERYING) — undefined

---

> 📘 *This lesson is part of the [PostgreSQL JSON & Document Processing](https://stanza.dev/courses/postgresql-json) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*