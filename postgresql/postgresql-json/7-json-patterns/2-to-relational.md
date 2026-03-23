---
source_course: "postgresql-json"
source_lesson: "postgresql-json-to-relational"
---

# Converting JSON to Relational Tables

## Introduction
PostgreSQL provides functions to expand JSON data into relational form for analysis, reporting, and integration with BI tools. These functions bridge the gap between document and relational models, letting you store data as JSON but query it as tables when needed.

## Key Concepts
- **json_populate_record()**: Fills a PostgreSQL record type from a JSON object, mapping keys to column names.
- **json_to_record()**: Creates an ad-hoc record from JSON without requiring a predefined type.
- **jsonb_array_elements()**: A set-returning function that expands a JSON array into one row per element.
- **WITH ORDINALITY**: An option that adds a row number column to set-returning function output.

## Real World Context
BI tools like Metabase and Tableau work best with flat, relational data. When your application stores orders as JSON documents with nested line items, reporting requires flattening that structure. Creating views that use these conversion functions gives analysts clean tabular data while the application continues to work with the flexible JSON format.

## Deep Dive

### json_populate_record: Fill a Record Type

Map JSON keys to the fields of an existing type or table:

```sql
-- Define a type or use existing table structure
CREATE TYPE person AS (name text, age int, city text);

-- Populate from JSON
SELECT * FROM json_populate_record(
    null::person,
    '{"name": "Alice", "age": 30, "city": "NYC"}'
);
-- Result: (Alice, 30, NYC)
```

The function matches JSON keys to record field names. Extra keys in the JSON are ignored, and missing keys become NULL.

### json_to_record: Ad-hoc Conversion

For one-off conversions without a predefined type:

```sql
-- No predefined type needed
SELECT * FROM json_to_record(
    '{"a": 1, "b": "hello", "c": true}'
) AS x(a int, b text, c boolean);
-- Result: (1, hello, true)
```

You define the output columns inline. This is convenient for ad-hoc queries.

### Expanding Arrays to Rows

The `jsonb_array_elements` function is the workhorse for flattening:

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
```

The `WITH ORDINALITY` clause adds a row number, which is useful for preserving array order.

### Practical: Expand Nested JSON to Rows

Combine array expansion with extraction for flattened output:

```sql
SELECT 
    o.id AS order_id,
    o.created_at,
    (item->>'product_id')::int AS product_id,
    (item->>'quantity')::int AS quantity,
    (item->>'price')::numeric AS price
FROM orders o,
    jsonb_array_elements(o.data->'items') AS item;
```

The implicit LATERAL join expands each order's items array into separate rows.

### Creating Views for Reporting

Wrap conversion logic in views for reusable access:

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

BI tools can query this view as if it were a regular table.

## Common Pitfalls
1. **Performance of array expansion on large tables** — `jsonb_array_elements` creates one row per element, which can multiply the result set dramatically. Always filter with WHERE before expanding.
2. **NULL handling with jsonb_array_elements** — If the array column is NULL or the path does not exist, `jsonb_array_elements` returns zero rows, silently dropping the parent row. Use LEFT JOIN LATERAL to preserve parent rows.
3. **Type casting errors** — If any element has an unexpected type (e.g., a string where you cast to int), the entire query fails. Add error handling or filter invalid rows.

## Best Practices
1. **Create views for common flattening patterns** — Views make JSON-to-relational conversion reusable and maintainable.
2. **Use LEFT JOIN LATERAL to preserve parent rows** — When some rows might have empty or NULL arrays, lateral joins prevent data loss.
3. **Consider JSON_TABLE on PG17+** — For complex flattening, `JSON_TABLE` is cleaner than combining multiple functions.

## Summary
- `json_populate_record` and `json_to_record` convert JSON objects to typed records.
- `jsonb_array_elements` expands arrays to rows, essential for flattening nested data.
- `WITH ORDINALITY` preserves array order with row numbers.
- Wrap conversions in views for BI tool access.
- Use LEFT JOIN LATERAL when arrays might be NULL or empty.

## Code Examples

**JSON to Relational Conversion**

```sql
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