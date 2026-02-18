---
source_course: "postgresql-json"
source_lesson: "generated-columns-json"
---

# Generated Columns from JSON

PostgreSQL 12+ supports generated columns, which can extract and compute values from JSONB columns automatically.

## Generated Columns Basics

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

## Benefits of Generated Columns

1. **Automatic extraction**: No need to update multiple columns
2. **Indexable**: Create B-tree indexes on generated columns
3. **Queryable**: Use regular SQL operators
4. **Consistent**: Always in sync with source JSON

```sql
-- Index on generated column
CREATE INDEX idx_products_name ON products (name);
CREATE INDEX idx_products_price ON products (price);

-- Efficient queries
SELECT * FROM products WHERE name ILIKE '%iphone%';
SELECT * FROM products WHERE price BETWEEN 500 AND 1000;
```

## Practical Pattern

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

## Code Examples

```undefined
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