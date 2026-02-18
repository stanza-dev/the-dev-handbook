---
source_course: "postgresql-json"
source_lesson: "creating-json-data"
---

# Creating and Storing JSON Data

There are multiple ways to create JSON/JSONB values in PostgreSQL.

## Casting Strings to JSON

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

## Building JSON Objects

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

## Aggregating to JSON

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

## Table with JSONB Column

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

## Code Examples

```undefined
-- Build complex JSON from query data
SELECT jsonb_build_object(
    'order_id', o.id,
    'customer', jsonb_build_object(
        'name', c.name,
        'email', c.email
    ),
    'items', (
        SELECT jsonb_agg(jsonb_build_object(
            'product', p.name,
            'quantity', oi.quantity,
            'price', oi.unit_price
        ))
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