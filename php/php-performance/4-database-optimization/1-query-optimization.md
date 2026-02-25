---
source_course: "php-performance"
source_lesson: "php-performance-query-optimization"
---

# Query Optimization Techniques

## Introduction
Database queries are often the biggest performance bottleneck.

## Key Concepts
- **EXPLAIN**: MySQL command that shows how a query will be executed — table scans, index usage, sort operations.
- **Index Selectivity**: How effectively an index narrows down results. High selectivity (few matching rows) = fast queries.
- **type=ALL in EXPLAIN**: Indicates a full table scan — the most expensive access type that should be eliminated.
- **Covering Index**: An index that contains all columns needed by the query, avoiding table lookups entirely.

## Real World Context
Database queries are the #1 performance bottleneck in most PHP applications. A single unindexed query on a million-row table can take 5+ seconds. EXPLAIN is the essential diagnostic tool — Facebook, Instagram, and Slack all use it extensively to optimize their MySQL queries.

## Deep Dive
### Intro

Database queries are often the biggest performance bottleneck.

### Indexing strategy

```sql
-- Single column index
CREATE INDEX idx_users_email ON users(email);

-- Composite index (order matters!)
CREATE INDEX idx_orders_user_status ON orders(user_id, status);

-- Covering index (includes all needed columns)
CREATE INDEX idx_users_search ON users(email, name, created_at);

-- Partial index (PostgreSQL)
CREATE INDEX idx_active_users ON users(email) WHERE status = 'active';
```

### Explain analysis

```php
<?php
function explainQuery(PDO $pdo, string $sql): array
{
    $stmt = $pdo->query('EXPLAIN ' . $sql);
    return $stmt->fetchAll();
}

// Look for:
// - type: 'ALL' (full table scan - bad)
// - type: 'ref', 'eq_ref', 'const' (using index - good)
// - rows: Lower is better
// - Extra: 'Using index' (covering index - excellent)
```

### Avoiding full table scans

```php
<?php
// BAD: Function on indexed column prevents index use
$pdo->query('SELECT * FROM users WHERE YEAR(created_at) = 2024');

// GOOD: Range query uses index
$pdo->query('SELECT * FROM users WHERE created_at >= "2024-01-01" AND created_at < "2025-01-01"');

// BAD: Leading wildcard
$pdo->query('SELECT * FROM users WHERE email LIKE "%@gmail.com"');

// GOOD: Trailing wildcard can use index
$pdo->query('SELECT * FROM users WHERE email LIKE "john%"');

// BAD: Implicit type conversion
$pdo->query('SELECT * FROM users WHERE id = "123"');  // id is INT

// GOOD: Correct type
$pdo->query('SELECT * FROM users WHERE id = 123');
```

### Select only what you need

```php
<?php
// BAD: Select everything
$pdo->query('SELECT * FROM users');

// GOOD: Select only needed columns
$pdo->query('SELECT id, name, email FROM users');

// Even better with covering index
// Index: (status, id, name)
$pdo->query('SELECT id, name FROM users WHERE status = "active"');
// Can be satisfied entirely from index!
```

### Limit results

```php
<?php
// BAD: Fetch all, then limit in PHP
$allUsers = $pdo->query('SELECT * FROM users')->fetchAll();
$pageUsers = array_slice($allUsers, 0, 20);

// GOOD: Limit in SQL
$pdo->query('SELECT * FROM users ORDER BY id LIMIT 20 OFFSET 0');

// Better for deep pagination: Cursor-based
$pdo->query('SELECT * FROM users WHERE id > :lastId ORDER BY id LIMIT 20');
```

### Batch operations

```php
<?php
// BAD: Many individual inserts
foreach ($users as $user) {
    $pdo->prepare('INSERT INTO users (name, email) VALUES (?, ?)')
        ->execute([$user['name'], $user['email']]);
}

// GOOD: Batch insert
$values = [];
$params = [];
foreach ($users as $i => $user) {
    $values[] = "(:name$i, :email$i)";
    $params["name$i"] = $user['name'];
    $params["email$i"] = $user['email'];
}

$sql = 'INSERT INTO users (name, email) VALUES ' . implode(', ', $values);
$pdo->prepare($sql)->execute($params);
```

## Common Pitfalls
1. **Adding indexes without analyzing EXPLAIN output** — Random indexes may not help and slow down writes. Always verify with EXPLAIN that your new index is actually used.
2. **Indexing low-selectivity columns** — Indexing a boolean column (e.g., `is_active`) is nearly useless because it doesn't narrow results. Index columns with high cardinality.

## Best Practices
1. **Run EXPLAIN on every slow query** — Identify `type=ALL` (full table scan), `Using filesort` (in-memory sort), and `Using temporary` (temp table creation).
2. **Create composite indexes matching your WHERE + ORDER BY** — A single composite index on `(status, created_at)` is more effective than two separate indexes.

## Summary
- Use EXPLAIN to analyze query execution plans and identify full table scans and unnecessary sorts.
- Create composite indexes matching your most common WHERE and ORDER BY patterns.
- Focus on high-selectivity columns and covering indexes for maximum query performance.

## Resources

- [MySQL EXPLAIN](https://dev.mysql.com/doc/refman/8.4/en/explain.html) — MySQL EXPLAIN statement documentation

---

> 📘 *This lesson is part of the [PHP Performance Optimization](https://stanza.dev/courses/php-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*