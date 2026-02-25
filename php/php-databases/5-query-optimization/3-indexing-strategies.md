---
source_course: "php-databases"
source_lesson: "php-databases-indexing-strategies"
---

# Indexing Strategies for PHP Applications

## Introduction

Indexes are the most powerful tool for query optimization. They transform full table scans into fast lookups, but they also consume storage and slow down writes. Choosing the right indexes requires understanding how your application queries data.

## Key Concepts

- **B-Tree Index**: The default index type. Supports equality, range, and prefix lookups. Organized as a balanced tree for O(log n) lookups.
- **Composite Index**: An index on multiple columns. The leftmost prefix rule determines which queries can use it.
- **Covering Index**: An index that contains all columns needed by a query, so the database never reads the actual table rows.

## Real World Context

A product catalog filtering by category, price range, and availability needs a composite index designed for exactly that query pattern. The wrong column order in the index means it gets ignored entirely, falling back to a full table scan on millions of rows.

## Deep Dive

The leftmost prefix rule is the most important concept for composite indexes:

```sql
CREATE INDEX idx_orders ON orders(user_id, status, created_at);

-- Uses index (matches leftmost prefix):
WHERE user_id = 1
WHERE user_id = 1 AND status = 'active'
WHERE user_id = 1 AND status = 'active' AND created_at > '2024-01-01'

-- Does NOT use index (skips user_id):
WHERE status = 'active'
WHERE created_at > '2024-01-01'
WHERE status = 'active' AND created_at > '2024-01-01'
```

Designing indexes from your query patterns:

```php
<?php
// Query pattern 1: Find user's recent orders
// SELECT * FROM orders WHERE user_id = ? ORDER BY created_at DESC
// Index: (user_id, created_at)

// Query pattern 2: Find pending orders
// SELECT * FROM orders WHERE status = 'pending' ORDER BY created_at
// Index: (status, created_at)

// Query pattern 3: Dashboard count by status for a user
// SELECT status, COUNT(*) FROM orders WHERE user_id = ? GROUP BY status
// Index: (user_id, status) — covering index for this query!
```

Covering indexes avoid table lookups entirely:

```sql
-- If index is (user_id, email)
SELECT email FROM users WHERE user_id = 5;
-- This is a "covering" query — the index has both columns
-- EXPLAIN shows Extra: Using index
```

Partial indexes (PostgreSQL) index only rows matching a condition:

```sql
-- PostgreSQL only: index only active users
CREATE INDEX idx_active_users ON users(email) WHERE status = 'active';
-- Smaller index, faster for queries that filter by status = 'active'
```

Index maintenance considerations:

```php
<?php
// Check index usage in MySQL
$stmt = $pdo->query('
    SELECT index_name, rows_read, rows_requested
    FROM sys.schema_index_statistics
    WHERE table_name = "orders"
');
// Remove indexes with zero reads — they slow down writes for no benefit
```

Every index slows down INSERT, UPDATE, and DELETE because the index must be updated too. Only create indexes that serve actual query patterns.

## Common Pitfalls

1. **Creating single-column indexes for every column** — This wastes storage and slows writes. Design composite indexes that match your actual query patterns.
2. **Wrong column order in composite indexes** — An index on `(status, user_id)` does not help a query filtering only by `user_id`. Put the most selective column first.

## Best Practices

1. **Design indexes from your query patterns, not your schema** — Look at your WHERE, ORDER BY, and GROUP BY clauses to determine which composite indexes to create.
2. **Monitor and remove unused indexes** — Use `sys.schema_index_statistics` in MySQL or `pg_stat_user_indexes` in PostgreSQL to find indexes that are never read.

## Summary

- Composite indexes follow the leftmost prefix rule: queries must filter on columns from left to right.
- Covering indexes contain all columns a query needs, eliminating table row lookups.
- Design indexes based on actual query patterns, not just schema structure.
- Monitor index usage and remove unused indexes to reduce write overhead.

## Code Examples

**Creating and verifying composite indexes**

```php
<?php
declare(strict_types=1);

// Create indexes matching actual query patterns
$pdo->exec('CREATE INDEX idx_orders_user_date ON orders(user_id, created_at)');
$pdo->exec('CREATE INDEX idx_orders_status_date ON orders(status, created_at)');
$pdo->exec('CREATE INDEX idx_products_category_price ON products(category_id, price)');

// Verify with EXPLAIN
$plan = $pdo->query(
    'EXPLAIN SELECT * FROM orders WHERE user_id = 1 ORDER BY created_at DESC LIMIT 10'
)->fetch();

echo "Access type: {$plan['type']}\n";  // Should be 'ref'
echo "Index used: {$plan['key']}\n";    // Should be 'idx_orders_user_date'
?>
```


## Resources

- [Use The Index, Luke](https://use-the-index-luke.com/) — Comprehensive guide to SQL indexing

---

> 📘 *This lesson is part of the [PHP & Relational Databases](https://stanza.dev/courses/php-databases) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*