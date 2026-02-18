---
source_course: "php-performance"
source_lesson: "php-performance-query-optimization"
---

# Query Optimization Techniques

Database queries are often the biggest performance bottleneck.

## Indexing Strategy

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

## EXPLAIN Analysis

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

## Avoiding Full Table Scans

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

## SELECT Only What You Need

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

## Limit Results

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

## Batch Operations

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

---

> 📘 *This lesson is part of the [PHP Performance Optimization](https://stanza.dev/courses/php-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*