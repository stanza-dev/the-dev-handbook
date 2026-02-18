---
source_course: "postgresql-json"
source_lesson: "json-to-relational"
---

# Converting JSON to Relational Tables

PostgreSQL provides functions to expand JSON data into relational form for analysis and reporting.

## json_populate_record: Fill a Record Type

```sql
-- Define a type or use existing table structure
CREATE TYPE person AS (name text, age int, city text);

-- Populate from JSON
SELECT * FROM json_populate_record(
    null::person,
    '{"name": "Alice", "age": 30, "city": "NYC"}'
);
-- Result: (Alice, 30, NYC)

-- Use with existing table type
SELECT * FROM json_populate_record(
    null::users,
    '{"email": "alice@test.com", "name": "Alice"}'
);
```

## json_to_record: Ad-hoc Conversion

```sql
-- No predefined type needed
SELECT * FROM json_to_record(
    '{"a": 1, "b": "hello", "c": true}'
) AS x(a int, b text, c boolean);
-- Result: (1, hello, true)
```

## Expanding Arrays to Rows

```sql
-- Expand JSON array
SELECT * FROM jsonb_array_elements(
    '[{"id": 1}, {"id": 2}, {"id": 3}]'::jsonb
);

-- With row numbers
SELECT 
    ordinality,
    value->>'id' AS id
FROM jsonb_array_elements('[{"id": 1}, {"id": 2}]'::jsonb) 
    WITH ORDINALITY;

-- Practical: Expand nested JSON to rows
SELECT 
    o.id AS order_id,
    o.created_at,
    (item->>'product_id')::int AS product_id,
    (item->>'quantity')::int AS quantity,
    (item->>'price')::numeric AS price
FROM orders o,
    jsonb_array_elements(o.data->'items') AS item;
```

## Creating Views for Reporting

```sql
-- Flatten JSON for BI tools
CREATE VIEW orders_flat AS
SELECT 
    o.id,
    o.created_at,
    o.data->>'customer_id' AS customer_id,
    o.data->>'status' AS status,
    (o.data->>'total')::numeric AS total,
    item.value->>'product' AS product,
    (item.value->>'quantity')::int AS quantity
FROM orders o,
    jsonb_array_elements(o.data->'items') AS item;
```

## Code Examples

```undefined
-- Create reporting view from JSON data
CREATE VIEW event_analytics AS
SELECT 
    id,
    (data->>'timestamp')::timestamptz AS event_time,
    data->>'type' AS event_type,
    data->>'user_id' AS user_id,
    data->'properties'->>'page' AS page,
    data->'properties'->>'referrer' AS referrer,
    (data->'properties'->>'duration')::int AS duration_ms
FROM events
WHERE data ? 'type';

-- Query the flattened view
SELECT 
    event_type,
    COUNT(*) AS event_count,
    AVG(duration_ms) AS avg_duration
FROM event_analytics
WHERE event_time > now() - interval '7 days'
GROUP BY event_type;
```


## Resources

- [JSON Processing Functions](https://www.postgresql.org/docs/current/functions-json.html#FUNCTIONS-JSON-PROCESSING) — undefined

---

> 📘 *This lesson is part of the [PostgreSQL JSON & Document Processing](https://stanza.dev/courses/postgresql-json) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*