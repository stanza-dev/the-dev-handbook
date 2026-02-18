---
source_course: "postgresql-performance"
source_lesson: "postgresql-performance-partial-expression-indexes"
---

# Partial and Expression Indexes

Advanced indexing techniques for specific workloads.

## Partial Indexes

Index only rows matching a condition:

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

**Benefits:**
- Smaller index size
- Faster to maintain
- Query must include the WHERE condition

```sql
-- Uses the partial index
SELECT * FROM users WHERE email = 'test@example.com' AND is_active = true;

-- Cannot use the partial index (missing is_active)
SELECT * FROM users WHERE email = 'test@example.com';
```

## Expression Indexes

Index the result of an expression:

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

**Important:** The query must use the exact same expression.

## Covering Indexes (INCLUDE)

Add non-key columns to enable Index Only Scans:

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

**Without INCLUDE:**
- Index Scan: Read index → Read heap for other columns

**With INCLUDE:**
- Index Only Scan: Read index only (faster!)

## Unique Partial Indexes

Enforce uniqueness only for subset:

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

## Real-World Example: E-commerce Search

```sql
-- Products table with various search patterns
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

📖 [Partial Indexes](https://www.postgresql.org/docs/18/indexes-partial.html)

## Resources

- [Expression Indexes](https://www.postgresql.org/docs/18/indexes-expressional.html) — Indexes on expressions

---

> 📘 *This lesson is part of the [PostgreSQL Performance Engineering](https://stanza.dev/courses/postgresql-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*