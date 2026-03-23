---
source_course: "postgresql-json"
source_lesson: "postgresql-json-expression-indexes"
---

# Expression Indexes on JSON

## Introduction
When you need to query specific JSON fields with equality or comparison operators (using `->>`), GIN indexes do not help. Expression indexes (also called functional indexes) solve this by indexing the extracted value directly. They create a B-tree index on the result of an expression, enabling fast lookups on specific JSON fields.

## Key Concepts
- **Expression index**: A B-tree index created on the result of a function or expression, such as `(data->>'name')`.
- **Type casting in indexes**: You can cast extracted values in the index definition, e.g., `((data->>'price')::numeric)`, enabling range queries.
- **Unique expression index**: Creates a uniqueness constraint on a value extracted from JSON, e.g., unique email from a JSONB column.
- **Index expression matching**: The query must use the exact same expression as the index for PostgreSQL to use it.

## Real World Context
Expression indexes are essential when your schema uses JSONB for flexibility but still needs fast lookups on specific fields. A SaaS application might store tenant-specific configuration in JSONB but need fast lookups by `data->>'tenant_id'`. Expression indexes give you the flexibility of JSONB with the query performance of dedicated columns.

## Deep Dive

### Creating Expression Indexes

Create an index on a specific extracted field:

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

Notice the double parentheses — the outer ones are required by CREATE INDEX syntax, and the inner ones wrap the expression.

### Unique Constraints on JSON Fields

Expression indexes can enforce uniqueness:

```sql
-- Unique constraint on JSON field
CREATE UNIQUE INDEX idx_users_email 
    ON users ((data->>'email'));

-- Partial unique index
CREATE UNIQUE INDEX idx_active_users_email 
    ON users ((data->>'email'))
    WHERE (data->>'status')::text = 'active';
```

This is powerful for enforcing business rules on flexible JSONB data.

### Combining Indexes

Multi-column expression indexes support compound queries:

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

The query planner uses the compound index when both conditions match the index columns.

## Common Pitfalls
1. **Expression mismatch** — The query expression must exactly match the index expression. `(data->>'price')::int` will not use an index on `(data->>'price')::numeric`.
2. **Forgetting parentheses** — Expression indexes require extra parentheses around the expression: `CREATE INDEX idx ON t ((expr))`.
3. **Creating too many expression indexes** — Each index adds write overhead. If you need many indexed fields, consider using generated columns instead.

## Best Practices
1. **Use expression indexes for frequently filtered specific fields** — They provide B-tree performance for specific JSONB field access.
2. **Use GIN indexes for flexible containment queries** — Do not create expression indexes for fields queried with `@>`.
3. **Consider generated columns for heavily indexed fields** — If you have more than 3-4 expression indexes on the same column, generated columns may be cleaner.

## Summary
- Expression indexes create B-tree indexes on extracted JSONB values for fast equality and range queries.
- The query expression must exactly match the index expression.
- Type casting is supported in index definitions for numeric range queries.
- Unique expression indexes enforce uniqueness constraints on JSON fields.
- Use expression indexes for specific field lookups and GIN indexes for containment queries.

## Code Examples

**Expression Index Examples**

```sql
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