---
source_course: "postgresql-performance"
source_lesson: "postgresql-performance-partial-expression-indexes"
---

# Partial and Expression Indexes

## Introduction

Partial and expression indexes are advanced indexing techniques that let you index subsets of data or computed values. They can dramatically reduce index size, speed up maintenance, and enable efficient queries that standard indexes cannot serve.

## Key Concepts

- **Partial index**: An index with a WHERE clause that only includes rows matching a condition.
- **Expression index**: An index on the result of a function or expression rather than a raw column value.
- **Covering index (INCLUDE)**: An index with additional non-key columns that enables Index Only Scans.
- **Unique partial index**: Enforces uniqueness only for rows matching a condition.

## Real World Context

In most applications, queries filter on specific subsets: active users, pending orders, published articles. A partial index on just those rows is smaller, faster to scan, and cheaper to maintain than a full index. Expression indexes are essential for case-insensitive searches and JSONB field queries.

## Deep Dive

### Partial Indexes

A partial index only includes rows matching its WHERE condition:

```sql
-- Only index active users (common query pattern)
CREATE INDEX idx_active_users 
ON users(email) 
WHERE is_active = true;

-- Only index unshipped orders
CREATE INDEX idx_pending_orders 
ON orders(created_at) 
WHERE status = 'pending';
```

These indexes are smaller and faster to maintain because they exclude irrelevant rows. However, the query must include the same WHERE condition to use the partial index:

```sql
-- Uses the partial index
SELECT * FROM users WHERE email = 'test@example.com' AND is_active = true;

-- Cannot use the partial index (missing is_active)
SELECT * FROM users WHERE email = 'test@example.com';
```

The planner matches the query's conditions against the index's predicate.

### Expression Indexes

Expression indexes index computed values rather than raw column data:

```sql
-- Index on lowercase email (case-insensitive search)
CREATE INDEX idx_users_email_lower ON users(LOWER(email));

-- Query must use the same expression
SELECT * FROM users WHERE LOWER(email) = 'alice@example.com';

-- Index on extracted JSONB field
CREATE INDEX idx_data_type ON documents((data->>'type'));

-- Index on date part of timestamp
CREATE INDEX idx_orders_date ON orders(DATE(created_at));
```

The query must use the exact same expression for the index to be used.

### Covering Indexes (INCLUDE)

The INCLUDE clause adds non-key columns to enable Index Only Scans:

```sql
-- Include columns needed by SELECT but not WHERE
CREATE INDEX idx_orders_covering 
ON orders(user_id) 
INCLUDE (total, created_at);

-- This can now be Index Only Scan
SELECT total, created_at 
FROM orders 
WHERE user_id = 42;
```

Without INCLUDE, PostgreSQL would need an Index Scan (read index then read heap). With INCLUDE, an Index Only Scan reads everything from the index.

### Unique Partial Indexes

Enforce uniqueness for a specific subset of rows:

```sql
-- Each user can have only one active subscription
CREATE UNIQUE INDEX idx_one_active_sub 
ON subscriptions(user_id) 
WHERE status = 'active';

-- Unique slug per published article (drafts can have duplicates)
CREATE UNIQUE INDEX idx_unique_published_slug 
ON articles(slug) 
WHERE published = true;
```

This is a powerful pattern for enforcing business rules at the database level.

### Real-World Example: E-commerce Search

Combining these techniques for a products table:

```sql
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name TEXT,
    category_id INT,
    price NUMERIC,
    is_available BOOLEAN,
    data JSONB
);

-- Index only available products by category
CREATE INDEX idx_available_by_category 
ON products(category_id, price) 
WHERE is_available = true;

-- Case-insensitive name search
CREATE INDEX idx_products_name_lower 
ON products(LOWER(name));

-- JSONB attribute search
CREATE INDEX idx_products_brand 
ON products((data->>'brand'));
```

Each index targets a specific query pattern with minimal overhead.

## Common Pitfalls

1. **Forgetting to match the partial index predicate** - If your query does not include the exact WHERE condition from the partial index definition, the planner cannot use that index.
2. **Expression mismatch** - `WHERE LOWER(email) = 'test'` uses the expression index, but `WHERE email = 'test'` does not. The expression must match exactly.
3. **Over-using INCLUDE** - Adding too many INCLUDE columns makes the index larger and slower to maintain. Only include columns that your most frequent queries actually need.

## Best Practices

1. **Use partial indexes for common filtered queries** - If 90% of your queries filter by `status = 'active'`, a partial index on active records is much more efficient than a full index.
2. **Create expression indexes for case-insensitive searches** - Use `LOWER()` or `citext` extension instead of ILIKE queries on large tables.
3. **Combine partial and expression indexes** - You can create `CREATE INDEX ... ON table(LOWER(name)) WHERE is_active = true` for maximum efficiency.

## Summary

- Partial indexes include only rows matching a WHERE condition, reducing size and maintenance cost.
- Expression indexes enable efficient queries on computed values like LOWER() or JSONB field extraction.
- INCLUDE columns in covering indexes enable Index Only Scans without bloating the index key.
- Unique partial indexes enforce business rules at the database level.
- The query must match the index's expression and predicate exactly for the planner to use it.

## Code Examples

**Four advanced indexing techniques: partial, expression, covering, and unique partial indexes**

```sql
-- Partial index: only active users
CREATE INDEX idx_active_users ON users(email)
WHERE is_active = true;

-- Expression index: case-insensitive search
CREATE INDEX idx_lower_email ON users(LOWER(email));

-- Covering index: enables Index Only Scan
CREATE INDEX idx_orders_covering ON orders(user_id)
INCLUDE (total, created_at);

-- Unique partial: one active subscription per user
CREATE UNIQUE INDEX idx_one_active_sub
ON subscriptions(user_id)
WHERE status = 'active';
```


## Resources

- [Partial Indexes](https://www.postgresql.org/docs/18/indexes-partial.html) — Creating and using partial indexes
- [Expression Indexes](https://www.postgresql.org/docs/18/indexes-expressional.html) — Indexes on expressions

---

> 📘 *This lesson is part of the [PostgreSQL Performance Engineering](https://stanza.dev/courses/postgresql-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*