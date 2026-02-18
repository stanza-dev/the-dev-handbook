---
source_course: "php-databases"
source_lesson: "php-databases-explain-analyze"
---

# Using EXPLAIN to Analyze Queries

EXPLAIN reveals how the database executes your queries, helping identify performance issues.

## Basic EXPLAIN

```php
<?php
$stmt = $pdo->query('EXPLAIN SELECT * FROM users WHERE email = "john@example.com"');
print_r($stmt->fetch());

// Output:
// id: 1
// select_type: SIMPLE
// table: users
// type: const  (good! using unique index)
// possible_keys: idx_email
// key: idx_email
// rows: 1
// Extra: NULL
```

## EXPLAIN Types (Best to Worst)

| Type | Meaning | Action |
|------|---------|--------|
| system/const | Single row | Perfect |
| eq_ref | One row per join | Good |
| ref | Multiple rows via index | Good |
| range | Index range scan | Acceptable |
| index | Full index scan | Consider optimization |
| ALL | Full table scan | Needs optimization! |

## Identifying Problems

```php
<?php
// BAD: Full table scan (type: ALL)
$stmt = $pdo->query('EXPLAIN SELECT * FROM orders WHERE YEAR(created_at) = 2024');
// Functions on columns prevent index usage!

// GOOD: Uses index (type: range)
$stmt = $pdo->query('EXPLAIN SELECT * FROM orders WHERE created_at >= "2024-01-01" AND created_at < "2025-01-01"');

// BAD: No index on status
$stmt = $pdo->query('EXPLAIN SELECT * FROM orders WHERE status = "pending"');
// key: NULL means no index used

// After adding index:
// CREATE INDEX idx_status ON orders(status);
// key: idx_status
```

## EXPLAIN ANALYZE (MySQL 8.0+)

```php
<?php
// Shows actual execution time
$stmt = $pdo->query('
    EXPLAIN ANALYZE 
    SELECT u.name, COUNT(o.id) as order_count
    FROM users u
    LEFT JOIN orders o ON u.id = o.user_id
    GROUP BY u.id
');

// Output includes actual time and rows processed
```

## Common Query Problems

```sql
-- Problem: SELECT *
SELECT * FROM users;  -- Fetches all columns
SELECT id, name, email FROM users;  -- Better: only needed columns

-- Problem: N+1 queries
FOREACH user { SELECT * FROM orders WHERE user_id = ? }  -- N queries!
SELECT * FROM orders WHERE user_id IN (1,2,3,4,5);  -- 1 query

-- Problem: Function on indexed column
WHERE LOWER(email) = 'john@example.com'  -- Can't use index
WHERE email = 'john@example.com'  -- Uses index (store lowercase)

-- Problem: LIKE with leading wildcard
WHERE name LIKE '%john%'  -- Full scan
WHERE name LIKE 'john%'  -- Can use index
```

## Resources

- [MySQL EXPLAIN](https://dev.mysql.com/doc/refman/8.0/en/explain.html) — MySQL EXPLAIN statement documentation

---

> 📘 *This lesson is part of the [PHP & Relational Databases](https://stanza.dev/courses/php-databases) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*