---
source_course: "postgresql-json"
source_lesson: "postgresql-json-jsonpath-variables"
---

# JSONPath Variables and Methods

## Introduction
Advanced JSONPath features include passing external variables into path expressions and calling methods on values for type introspection and conversion. Variables make expressions dynamic and reusable, while methods enable transformations directly within the path language.

## Key Concepts
- **Variables (`$varname`)**: Named parameters passed as a JSONB object in the third argument of path functions. They allow dynamic filtering without string concatenation.
- **`.type()` method**: Returns the JSON type of a value as a string (e.g., "object", "array", "number").
- **`.size()` method**: Returns the number of elements in a JSON array.
- **`.double()` method**: Converts a string value to a numeric double-precision value.
- **`.keyvalue()` method**: Expands a JSON object into key-value pair objects.

## Real World Context
In production, filter thresholds come from application parameters, not hardcoded values. Variables let you write a single JSONPath expression and pass different thresholds at runtime — for example, filtering products by a user-selected minimum price. Methods like `.size()` are useful for finding documents with arrays of a certain length, such as orders with more than 5 line items.

## Deep Dive

### Passing Variables to JSONPath

The third argument to JSONPath functions accepts a JSONB object of variable values:

```sql
-- Third argument passes variables as JSON object
SELECT jsonb_path_query_array(
    '[{"price": 100}, {"price": 200}, {"price": 150}]'::jsonb,
    '$[*] ? (@.price < $max_price)',
    '{"max_price": 175}'::jsonb
);
-- Result: [{"price": 100}, {"price": 150}]

-- Multiple variables
SELECT jsonb_path_query_array(
    '[{"price": 100, "qty": 5}, {"price": 200, "qty": 10}]'::jsonb,
    '$[*] ? (@.price >= $min && @.price <= $max)',
    '{"min": 50, "max": 150}'::jsonb
);
```

Variables are prefixed with `$` and referenced by name in the path expression. This approach is safe from injection.

### JSONPath Methods

Methods are called with dot notation on path values:

```sql
-- .type() returns JSON type
SELECT jsonb_path_query('{"name": "Alice", "age": 30}'::jsonb, '$.name.type()');
-- Result: "string"

-- .size() for arrays
SELECT jsonb_path_query('[1, 2, 3, 4, 5]'::jsonb, '$.size()');
-- Result: 5

-- .double() converts to number
SELECT jsonb_path_query('{"value": "42.5"}'::jsonb, '$.value.double()');
-- Result: 42.5

-- .ceiling(), .floor(), .abs()
SELECT jsonb_path_query('{"value": -3.7}'::jsonb, '$.value.abs().ceiling()');
-- Result: 4

-- .keyvalue() for object iteration
SELECT jsonb_path_query(
    '{"a": 1, "b": 2}'::jsonb,
    '$.keyvalue()'
);
-- Returns: {"key": "a", "value": 1}, {"key": "b", "value": 2}
```

Methods can be chained (`.abs().ceiling()`) and combined with filters for powerful expressions.

### Using JSONPath in WHERE Clauses

Here are practical patterns for filtering table data with JSONPath:

```sql
CREATE TABLE events (
    id SERIAL PRIMARY KEY,
    data JSONB
);

INSERT INTO events (data) VALUES
('{"type": "click", "page": "/home", "user_id": 1}'),
('{"type": "purchase", "amount": 99.99, "user_id": 2}'),
('{"type": "click", "page": "/products", "user_id": 1}');

-- Using jsonb_path_exists for filtering
SELECT * FROM events
WHERE jsonb_path_exists(data, '$ ? (@.type == "purchase")');

-- Using jsonb_path_match for boolean predicates
SELECT * FROM events
WHERE jsonb_path_match(data, '$.amount > 50');

-- With variables from application parameters
SELECT * FROM events
WHERE jsonb_path_exists(
    data,
    '$ ? (@.user_id == $uid)',
    jsonb_build_object('uid', 1)
);
```

These patterns work with GIN indexes for efficient large-scale filtering.

### Practical Example: Complex Query

Combining variables, methods, and filters:

```sql
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name TEXT,
    specs JSONB
);

INSERT INTO products (name, specs) VALUES
('Laptop A', '{"cpu": "Intel i7", "ram": 16, "storage": {"type": "SSD", "gb": 512}, "ports": ["USB-C", "HDMI"]}'),
('Laptop B', '{"cpu": "AMD Ryzen 9", "ram": 32, "storage": {"type": "SSD", "gb": 1024}, "ports": ["USB-C", "USB-A", "HDMI"]}');

-- Find laptops with >= 32GB RAM and SSD > 500GB
SELECT name, specs
FROM products
WHERE jsonb_path_exists(
    specs,
    '$ ? (@.ram >= $min_ram && @.storage.type == "SSD" && @.storage.gb > $min_storage)',
    '{"min_ram": 32, "min_storage": 500}'::jsonb
);
```

This single expression replaces what would otherwise require multiple extraction and comparison operations.

## Common Pitfalls
1. **Using $ for variables without passing the third argument** — If you reference `$min` in the path but do not pass a variables object, the query fails with an error.
2. **Calling .size() on a non-array** — In strict mode, calling `.size()` on an object or scalar raises an error. In lax mode, it returns NULL.
3. **Forgetting that .keyvalue() returns objects** — Each result is `{"key": ..., "value": ...}`, not a simple pair. Access fields with `.key` and `.value`.

## Best Practices
1. **Always use variables for dynamic values** — Never concatenate user input into path strings. Variables are safe and cleaner.
2. **Use .type() for defensive programming** — Check types before performing operations to avoid runtime errors in strict mode.
3. **Chain methods for complex transformations** — Combine `.double()`, `.abs()`, `.ceiling()` etc. to transform values within the path expression.

## Summary
- Variables are passed as a JSONB object in the third argument and referenced with `$name` in the path.
- Methods like `.type()`, `.size()`, `.double()`, and `.keyvalue()` transform and introspect values.
- Methods can be chained for multi-step transformations.
- Variables prevent injection and make path expressions reusable.
- Combine variables, methods, and filters for powerful single-expression queries.

## Code Examples

**JSONPath variables for dynamic filtering and methods for type introspection**

```sql
-- Pass variables as the third argument
SELECT jsonb_path_query_array(
    '[{"price": 100}, {"price": 200}, {"price": 150}]'::jsonb,
    '$[*] ? (@.price < $max_price)',
    '{"max_price": 175}'::jsonb
);
-- Result: [{"price": 100}, {"price": 150}]

-- .type() returns the JSON type
SELECT jsonb_path_query('{"name": "Alice"}'::jsonb, '$.name.type()');
-- Result: "string"

-- .size() counts array elements
SELECT jsonb_path_query('[1, 2, 3, 4, 5]'::jsonb, '$.size()');
-- Result: 5

-- .keyvalue() expands objects to key-value pairs
SELECT jsonb_path_query('{"a": 1, "b": 2}'::jsonb, '$.keyvalue()');
```


## Resources

- [JSONPath Reference](https://www.postgresql.org/docs/18/functions-json.html#FUNCTIONS-SQLJSON-PATH) — undefined

---

> 📘 *This lesson is part of the [PostgreSQL JSON & Document Processing](https://stanza.dev/courses/postgresql-json) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*