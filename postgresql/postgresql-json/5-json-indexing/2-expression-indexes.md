---
source_course: "postgresql-json"
source_lesson: "expression-indexes"
---

# Expression Indexes on JSON

When you need to query specific JSON fields with equality or comparison operators, expression indexes (functional indexes) are the solution.

## Creating Expression Indexes

```sql
-- Index on a specific text field
CREATE INDEX idx_products_brand 
    ON products ((attributes->>'brand'));

-- Now this query uses the index!
SELECT * FROM products 
WHERE attributes->>'brand' = 'Apple';

-- Index on numeric field (with cast)
CREATE INDEX idx_products_price 
    ON products (((attributes->>'price')::numeric));

SELECT * FROM products 
WHERE (attributes->>'price')::numeric < 100;

-- Index on nested field
CREATE INDEX idx_products_ram 
    ON products (((attributes->'specs'->>'ram')::int));
```

## Unique Constraints on JSON Fields

```sql
-- Unique constraint on JSON field
CREATE UNIQUE INDEX idx_users_email 
    ON users ((data->>'email'));

-- Partial unique index
CREATE UNIQUE INDEX idx_active_users_email 
    ON users ((data->>'email'))
    WHERE (data->>'status')::text = 'active';
```

## Combining Indexes

```sql
-- Multi-column expression index
CREATE INDEX idx_orders_customer_status 
    ON orders (
        (data->>'customer_id'),
        (data->>'status')
    );

-- Query benefits from compound index
SELECT * FROM orders 
WHERE data->>'customer_id' = '123' 
  AND data->>'status' = 'pending';
```

**Best Practice:** Use expression indexes for frequently queried specific fields, GIN for flexible containment queries.

## Code Examples

```undefined
-- Index specific JSON paths for common queries
CREATE INDEX idx_events_type ON events ((data->>'type'));
CREATE INDEX idx_events_timestamp ON events (((data->>'timestamp')::timestamptz));

-- Verify index usage
EXPLAIN ANALYZE
SELECT * FROM events 
WHERE data->>'type' = 'purchase'
  AND (data->>'timestamp')::timestamptz > '2024-01-01';

-- Composite index on JSON + regular columns
CREATE INDEX idx_orders_composite ON orders (
    created_at,
    (data->>'status')
);
```


## Resources

- [Indexes on Expressions](https://www.postgresql.org/docs/current/indexes-expressional.html) — undefined

---

> 📘 *This lesson is part of the [PostgreSQL JSON & Document Processing](https://stanza.dev/courses/postgresql-json) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*