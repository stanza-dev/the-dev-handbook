---
source_course: "postgresql-json"
source_lesson: "postgresql-json-generated-columns"
---

# Generated Columns from JSON

## Introduction
PostgreSQL 12+ supports generated columns, which automatically compute and store values derived from other columns. When combined with JSONB, generated columns extract frequently accessed fields into dedicated, indexable columns that stay in sync with the source JSON automatically. PostgreSQL 18 also introduces virtual generated columns for simple scalar expressions, though with some limitations on which types and operators they support.

## Key Concepts
- **Stored generated column**: A column whose value is computed from an expression and physically stored on disk. Updated automatically whenever the source column changes.
- **Virtual generated column (PG18)**: A column whose value is computed on-the-fly during reads, without occupying storage. Note: virtual generated columns cannot use JSONB operators or functions, so JSONB extraction still requires STORED generated columns.
- **GENERATED ALWAYS AS**: The SQL syntax for defining a generated column, followed by `STORED` or `VIRTUAL`.
- **Automatic sync**: Generated columns are always consistent with their source data because the database engine manages updates.

## Real World Context
When your application stores JSON documents but needs fast lookups on specific fields, generated columns provide a clean solution. Rather than maintaining expression indexes (which require exact expression matching in queries), generated columns create real columns that any query can use naturally. Analytics dashboards that need to GROUP BY or ORDER BY JSON fields benefit greatly from generated columns.

## Deep Dive

### Generated Columns Basics

Define generated columns that extract values from JSONB:

```sql
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    data JSONB NOT NULL,
    
    -- Generated columns from JSON
    name TEXT GENERATED ALWAYS AS (data->>'name') STORED,
    price NUMERIC GENERATED ALWAYS AS ((data->>'price')::numeric) STORED,
    brand TEXT GENERATED ALWAYS AS (data->>'brand') STORED
);

-- Insert only the JSON
INSERT INTO products (data) VALUES 
('{"name": "iPhone 15", "price": 999, "brand": "Apple"}');

-- Generated columns are automatically populated
SELECT id, name, price, brand FROM products;
-- Result: (1, 'iPhone 15', 999, 'Apple')
```

You only write to the `data` column; the generated columns populate themselves.

### Benefits of Generated Columns

Generated columns provide several advantages over expression indexes:

1. **Automatic extraction**: No need to update multiple columns manually
2. **Indexable**: Create standard B-tree indexes on generated columns
3. **Queryable**: Use regular SQL operators without expression matching
4. **Consistent**: Always in sync with source JSON

```sql
-- Index on generated column
CREATE INDEX idx_products_name ON products (name);
CREATE INDEX idx_products_price ON products (price);

-- Efficient queries — no expression matching needed
SELECT * FROM products WHERE name ILIKE '%iphone%';
SELECT * FROM products WHERE price BETWEEN 500 AND 1000;
```

Unlike expression indexes, queries do not need to use the exact same expression.

### Practical Pattern

A common production pattern for API-driven applications:

```sql
CREATE TABLE orders (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    raw_data JSONB NOT NULL,  -- Original API payload
    
    -- Extract common query fields
    customer_id TEXT GENERATED ALWAYS AS (raw_data->>'customer_id') STORED,
    status TEXT GENERATED ALWAYS AS (raw_data->>'status') STORED,
    total NUMERIC GENERATED ALWAYS AS ((raw_data->>'total')::numeric) STORED,
    created_at TIMESTAMPTZ GENERATED ALWAYS AS 
        ((raw_data->>'created_at')::timestamptz) STORED
);

-- Efficient filtering on extracted fields
CREATE INDEX idx_orders_customer ON orders (customer_id);
CREATE INDEX idx_orders_status ON orders (status);
CREATE INDEX idx_orders_created ON orders (created_at);
```

This pattern preserves the full JSON payload while providing fast relational access to key fields.

## Common Pitfalls
1. **Trying to write to generated columns** — Generated columns are read-only. INSERT and UPDATE statements cannot set their values directly.
2. **Using volatile functions in generated columns** — The expression must be immutable. Functions like `now()` or `random()` are not allowed.
3. **Excessive generated columns** — Each STORED generated column occupies disk space. Extract only the fields you actually need to query.

## Best Practices
1. **Use generated columns for 3+ frequently queried fields** — For one or two fields, expression indexes are simpler. For many fields, generated columns are cleaner.
2. **Keep the original JSON column** — Always preserve the full JSON document alongside generated columns for flexibility.
3. **Always use STORED for JSONB extraction** — PG18 virtual generated columns cannot use JSONB operators (`->`, `->>`) because `jsonb` is not a built-in base type in this context. For JSONB fields, always use `STORED`.

## Summary
- Generated columns automatically extract and store values from JSONB, staying in sync with the source.
- They support standard B-tree indexes and natural SQL query syntax.
- PostgreSQL 18 introduces virtual generated columns, but they cannot use JSONB operators — always use STORED for JSONB extraction.
- Use generated columns when you have 3+ frequently queried JSON fields.
- Generated column expressions must be immutable; volatile functions are not allowed.

## Code Examples

**Generated Columns Pattern**

```sql
-- Event tracking with generated columns
CREATE TABLE events (
    id BIGSERIAL PRIMARY KEY,
    payload JSONB NOT NULL,
    
    -- Generated columns for common queries
    event_type TEXT GENERATED ALWAYS AS (payload->>'type') STORED,
    user_id UUID GENERATED ALWAYS AS ((payload->>'user_id')::uuid) STORED,
    occurred_at TIMESTAMPTZ GENERATED ALWAYS AS 
        ((payload->>'timestamp')::timestamptz) STORED
);

-- Indexes on generated columns
CREATE INDEX idx_events_type ON events (event_type);
CREATE INDEX idx_events_user ON events (user_id);
CREATE INDEX idx_events_time ON events (occurred_at DESC);

-- Simple queries on complex JSON
SELECT * FROM events 
WHERE event_type = 'purchase' 
  AND occurred_at > now() - interval '1 hour';
```


## Resources

- [Generated Columns](https://www.postgresql.org/docs/current/ddl-generated-columns.html) — undefined

---

> 📘 *This lesson is part of the [PostgreSQL JSON & Document Processing](https://stanza.dev/courses/postgresql-json) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*