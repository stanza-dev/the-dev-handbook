---
source_course: "postgresql-json"
source_lesson: "jsonpath-variables"
---

# JSONPath Variables and Methods

Advanced JSONPath features for dynamic queries and data transformation.

## Passing Variables to JSONPath

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

## JSONPath Methods

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

## Using JSONPath in WHERE Clauses

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

-- With variables from other columns or parameters
SELECT * FROM events
WHERE jsonb_path_exists(
    data,
    '$ ? (@.user_id == $uid)',
    jsonb_build_object('uid', 1)
);
```

## Practical Example: Complex Query

```sql
-- Products table with JSON specs
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name TEXT,
    specs JSONB
);

INSERT INTO products (name, specs) VALUES
('Laptop A', '{"cpu": "Intel i7", "ram": 16, "storage": {"type": "SSD", "gb": 512}, "ports": ["USB-C", "HDMI"]}'),
('Laptop B', '{"cpu": "AMD Ryzen 9", "ram": 32, "storage": {"type": "SSD", "gb": 1024}, "ports": ["USB-C", "USB-A", "HDMI"]}');

-- Find laptops with >= 32GB RAM and SSD storage > 500GB
SELECT name, specs
FROM products
WHERE jsonb_path_exists(
    specs,
    '$ ? (@.ram >= $min_ram && @.storage.type == "SSD" && @.storage.gb > $min_storage)',
    '{"min_ram": 32, "min_storage": 500}'::jsonb
);
```

📖 [JSONPath Items](https://www.postgresql.org/docs/18/functions-json.html#FUNCTIONS-SQLJSON-PATH)

## Resources

- [JSONPath Reference](https://www.postgresql.org/docs/18/functions-json.html#FUNCTIONS-SQLJSON-PATH) — undefined

---

> 📘 *This lesson is part of the [PostgreSQL JSON & Document Processing](https://stanza.dev/courses/postgresql-json) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*