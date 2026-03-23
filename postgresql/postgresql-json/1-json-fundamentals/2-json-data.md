---
source_course: "postgresql-json"
source_lesson: "postgresql-json-creating-json-data"
---

# Creating and Storing JSON Data

## Introduction
PostgreSQL offers multiple ways to create JSON and JSONB values, from simple string casting to powerful construction functions that build nested structures from query results. Mastering these techniques lets you assemble complex documents directly in SQL.

## Key Concepts
- **Casting**: Converting a text string to `json` or `jsonb` using the `::jsonb` syntax.
- **jsonb_build_object()**: A function that constructs a JSONB object from alternating key-value argument pairs.
- **jsonb_build_array()**: A function that constructs a JSONB array from its arguments.
- **jsonb_agg() / jsonb_object_agg()**: Aggregate functions that collect rows into a JSON array or object.

## Real World Context
When building REST APIs, you often need to assemble JSON responses from normalized relational data. Rather than fetching rows and assembling JSON in application code, PostgreSQL can do this directly. This reduces round trips and simplifies your API layer. E-commerce platforms frequently use `jsonb_agg` to build order summaries with nested line items in a single query.

## Deep Dive

### Casting Strings to JSON

The simplest way to create JSON is by casting a string literal:

```sql
-- Direct casting
SELECT '{"name": "Alice", "age": 30}'::jsonb;

-- Using to_jsonb function
SELECT to_jsonb('hello');
-- Result: "hello"

-- From row
SELECT to_jsonb(row('Alice', 30));
-- Result: {"f1": "Alice", "f2": 30}
```

Casting is straightforward but requires you to write valid JSON strings manually.

### Building JSON Objects Programmatically

The `jsonb_build_object` function is safer than string casting because PostgreSQL handles escaping and type conversion automatically:

```sql
-- Build object from key-value pairs
SELECT jsonb_build_object(
    'name', 'Alice',
    'age', 30,
    'active', true
);
-- Result: {"age": 30, "name": "Alice", "active": true}

-- Build array
SELECT jsonb_build_array(1, 2, 'three', true);
-- Result: [1, 2, "three", true]

-- Nested structures
SELECT jsonb_build_object(
    'user', jsonb_build_object('name', 'Bob', 'id', 123),
    'roles', jsonb_build_array('admin', 'user')
);
```

Nesting these functions lets you build arbitrarily deep document structures.

### Aggregating to JSON

Aggregate functions let you collect multiple rows into a single JSON value:

```sql
-- Aggregate rows to JSON array
SELECT jsonb_agg(name) FROM users;
-- Result: ["Alice", "Bob", "Charlie"]

-- Aggregate to JSON object
SELECT jsonb_object_agg(id, name) FROM users;
-- Result: {"1": "Alice", "2": "Bob"}

-- Aggregate entire rows
SELECT jsonb_agg(to_jsonb(users)) FROM users;
```

These aggregations are invaluable for building nested API responses in a single query.

### Table with JSONB Column

Here is a typical table definition that combines relational columns with a JSONB attributes column:

```sql
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    attributes JSONB DEFAULT '{}',
    metadata JSONB
);

INSERT INTO products (name, attributes) VALUES
('Laptop', '{"brand": "Dell", "specs": {"ram": 16, "storage": 512}}'),
('Phone', '{"brand": "Apple", "model": "iPhone 15"}');
```

The `DEFAULT '{}'` ensures the column always has a valid empty object even when no attributes are provided.

## Common Pitfalls
1. **Forgetting to escape special characters in cast strings** — Manual casting can fail on strings containing quotes or backslashes. Use `jsonb_build_object` instead.
2. **Using jsonb_agg on empty result sets** — When no rows match, `jsonb_agg` returns NULL rather than an empty array. Wrap with `COALESCE(jsonb_agg(...), '[]'::jsonb)`.
3. **Passing an odd number of arguments to jsonb_build_object** — This function requires pairs (key, value). An odd count causes a runtime error.

## Best Practices
1. **Prefer jsonb_build_object over string casting** — It handles escaping, type conversion, and is less error-prone than constructing JSON strings manually.
2. **Use COALESCE with jsonb_agg** — Always wrap aggregations with `COALESCE(..., '[]'::jsonb)` to avoid NULL results on empty sets.
3. **Set meaningful defaults** — Use `DEFAULT '{}'` on JSONB columns so you never have to handle NULL checks in application code.

## Summary
- Cast strings with `::jsonb` for simple literals.
- Use `jsonb_build_object()` and `jsonb_build_array()` for safe, programmatic JSON construction.
- Aggregate rows into JSON arrays or objects with `jsonb_agg()` and `jsonb_object_agg()`.
- Always set `DEFAULT '{}'` on JSONB columns to avoid NULL handling.
- Wrap aggregations with COALESCE to handle empty result sets.

## Code Examples

**JSON Construction Functions**

```sql
-- Build complex JSON from query data
SELECT jsonb_build_object(
    'order_id', o.id,
    'customer', jsonb_build_object(
        'name', c.name,
        'email', c.email
    ),
    'items', (
        SELECT COALESCE(jsonb_agg(jsonb_build_object(
            'product', p.name,
            'quantity', oi.quantity,
            'price', oi.unit_price
        )), '[]'::jsonb)
        FROM order_items oi
        JOIN products p ON oi.product_id = p.id
        WHERE oi.order_id = o.id
    )
)
FROM orders o
JOIN customers c ON o.customer_id = c.id
WHERE o.id = 1;
```


## Resources

- [JSON Functions](https://www.postgresql.org/docs/current/functions-json.html) — undefined

---

> 📘 *This lesson is part of the [PostgreSQL JSON & Document Processing](https://stanza.dev/courses/postgresql-json) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*