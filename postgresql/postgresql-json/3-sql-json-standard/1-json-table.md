---
source_course: "postgresql-json"
source_lesson: "postgresql-json-json-table"
---

# JSON_TABLE: Converting JSON to Relational Rows

## Introduction
`JSON_TABLE` is a SQL-standard function introduced in PostgreSQL 17 that transforms JSON data into a relational table format. It replaces complex combinations of `jsonb_array_elements`, `jsonb_to_record`, and lateral joins with a single, declarative expression. This is one of the most powerful additions for working with JSON data.

## Key Concepts
- **JSON_TABLE()**: A SQL-standard function that maps a JSON document to a virtual relational table with typed columns.
- **COLUMNS clause**: Defines the output columns, their types, and the JSON path that populates each one.
- **NESTED PATH**: Allows flattening nested arrays within the same JSON_TABLE call, producing one row per nested element.
- **Error handling clauses**: `DEFAULT ... ON ERROR` and `NULL ON ERROR` control behavior when a JSON path fails or a type cast is invalid.

## Real World Context
Before `JSON_TABLE`, extracting structured data from complex JSON required verbose lateral joins with `jsonb_array_elements` and manual type casting. Reporting tools and ETL pipelines benefit enormously from `JSON_TABLE` because it produces clean, typed tabular output from nested documents in a single query. Data analysts working with JSON API responses can now write straightforward SELECT statements.

## Deep Dive

### Basic JSON_TABLE Usage

`JSON_TABLE` takes a JSON expression, a row path, and a COLUMNS definition:

```sql
SELECT jt.*
FROM JSON_TABLE(
    '[{"id": 1, "name": "Alice", "age": 30},
      {"id": 2, "name": "Bob", "age": 25}]'::jsonb,
    '$[*]'
    COLUMNS (
        id INTEGER PATH '$.id',
        name TEXT PATH '$.name',
        age INTEGER PATH '$.age'
    )
) AS jt;
```

This produces a clean relational result with typed columns. The `'$[*]'` path iterates over each array element, and each COLUMNS entry extracts a specific field.

### Using JSON_TABLE with Table Data

In practice, you use `JSON_TABLE` with data stored in a table column:

```sql
CREATE TABLE api_responses (
    id SERIAL PRIMARY KEY,
    payload JSONB NOT NULL
);

INSERT INTO api_responses (payload) VALUES
('{"orders": [{"oid": 101, "total": 59.99, "status": "shipped"},
              {"oid": 102, "total": 124.50, "status": "pending"}]}');

SELECT r.id AS response_id, jt.*
FROM api_responses r,
    JSON_TABLE(
        r.payload,
        '$.orders[*]'
        COLUMNS (
            order_id INTEGER PATH '$.oid',
            total NUMERIC PATH '$.total',
            status TEXT PATH '$.status'
        )
    ) AS jt;
```

This flattens the nested orders array into relational rows joined with the parent response.

### Nested Paths for Multi-Level Flattening

The `NESTED PATH` clause handles arrays within arrays:

```sql
SELECT jt.*
FROM JSON_TABLE(
    '{"customer": "Alice", "orders": [
        {"id": 1, "items": [{"product": "Laptop", "qty": 1}, {"product": "Mouse", "qty": 2}]},
        {"id": 2, "items": [{"product": "Keyboard", "qty": 1}]}
    ]}'::jsonb,
    '$.orders[*]'
    COLUMNS (
        order_id INTEGER PATH '$.id',
        NESTED PATH '$.items[*]'
        COLUMNS (
            product TEXT PATH '$.product',
            quantity INTEGER PATH '$.qty'
        )
    )
) AS jt;
```

This produces one row per item, with the order_id repeated for each item in that order. Without `JSON_TABLE`, this would require multiple lateral joins.

### Error Handling

`JSON_TABLE` supports error handling clauses:

```sql
SELECT jt.*
FROM JSON_TABLE(
    '[{"id": 1, "price": "29.99"}, {"id": 2, "price": "invalid"}]'::jsonb,
    '$[*]'
    COLUMNS (
        id INTEGER PATH '$.id',
        price NUMERIC PATH '$.price' DEFAULT 0 ON ERROR
    )
) AS jt;
```

The `DEFAULT 0 ON ERROR` clause returns 0 instead of failing when `"invalid"` cannot be cast to NUMERIC.

## Common Pitfalls
1. **Forgetting the alias** — `JSON_TABLE` requires an alias (e.g., `AS jt`). Omitting it causes a syntax error.
2. **Using wrong path for nested arrays** — The row path must point to the array level you want to iterate. `'$'` instead of `'$[*]'` returns a single row with the whole document.
3. **Implicit json-to-jsonb conversion overhead** — `JSON_TABLE` accepts both `json` and `jsonb`, but `json` input is implicitly cast to `jsonb` on every call. For best performance, store data as `jsonb` to avoid repeated conversion.

## Best Practices
1. **Use JSON_TABLE for complex flattening** — When you need typed columns from nested JSON arrays, `JSON_TABLE` is cleaner than lateral joins with `jsonb_array_elements`.
2. **Add error handling for external data** — When processing untrusted JSON (API responses, user uploads), use `DEFAULT ... ON ERROR` or `NULL ON ERROR` to handle malformed values gracefully.
3. **Create views for reusable transformations** — Wrap `JSON_TABLE` queries in views so reporting tools can treat JSON data as regular tables.

## Summary
- `JSON_TABLE` converts JSON documents into typed relational rows in a single expression.
- The COLUMNS clause maps JSON paths to output columns with specific SQL types.
- NESTED PATH handles multi-level array flattening without manual lateral joins.
- Error handling clauses (`DEFAULT ... ON ERROR`, `NULL ON ERROR`) make it safe for untrusted data.
- Available in PostgreSQL 17+ and replaces verbose `jsonb_array_elements` patterns.

## Code Examples

**JSON_TABLE with Nested Paths**

```sql
-- Flatten orders with line items in a single query
SELECT jt.*
FROM orders o,
    JSON_TABLE(
        o.data,
        '$.line_items[*]'
        COLUMNS (
            product_name TEXT PATH '$.name',
            quantity INTEGER PATH '$.qty',
            unit_price NUMERIC PATH '$.price',
            NESTED PATH '$.discounts[*]'
            COLUMNS (
                discount_code TEXT PATH '$.code',
                discount_pct NUMERIC PATH '$.percent'
            )
        )
    ) AS jt
WHERE o.status = 'pending';
```


## Resources

- [JSON_TABLE](https://www.postgresql.org/docs/current/functions-json.html#FUNCTIONS-SQLJSON-TABLE) — undefined

---

> 📘 *This lesson is part of the [PostgreSQL JSON & Document Processing](https://stanza.dev/courses/postgresql-json) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*