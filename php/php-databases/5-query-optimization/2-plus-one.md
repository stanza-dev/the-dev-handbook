---
source_course: "php-databases"
source_lesson: "php-databases-n-plus-one"
---

# Solving N+1 Query Problems

## Introduction

The N+1 problem is the most common performance mistake in database-driven applications. It occurs when code executes one query to fetch a list, then N individual queries to fetch related data for each item. The fix is always to batch the related data loading.

## Key Concepts

- **N+1 Problem**: Executing 1 query for a list plus N queries for related data, resulting in N+1 total queries.
- **Eager Loading**: Fetching all related data upfront in one or two queries instead of lazily loading it per item.
- **Batch Loading with IN**: Using `WHERE foreign_key IN (id1, id2, ...)` to load all related rows in a single query.

## Real World Context

A page showing 50 users with their latest orders could execute 51 queries (1 for users + 50 for orders). With eager loading, it executes exactly 2 queries regardless of how many users you display. At scale, this difference is the difference between 50ms and 5000ms page loads.

## Deep Dive

The N+1 problem in action:

```php
<?php
// 1 query to get users
$users = $pdo->query('SELECT * FROM users LIMIT 100')->fetchAll();

foreach ($users as $user) {
    // 100 more queries!
    $stmt = $pdo->prepare('SELECT * FROM orders WHERE user_id = ?');
    $stmt->execute([$user['id']]);
    $orders = $stmt->fetchAll();
}
// Total: 101 queries!
```

Solution 1 — JOIN (single query):

```php
<?php
$stmt = $pdo->query('
    SELECT u.*, o.id as order_id, o.total, o.created_at as order_date
    FROM users u
    LEFT JOIN orders o ON u.id = o.user_id
    ORDER BY u.id, o.created_at
');

$results = $stmt->fetchAll();

// Group by user in PHP
$users = [];
foreach ($results as $row) {
    $userId = $row['id'];
    if (!isset($users[$userId])) {
        $users[$userId] = [
            'id' => $row['id'],
            'name' => $row['name'],
            'orders' => [],
        ];
    }
    if ($row['order_id']) {
        $users[$userId]['orders'][] = [
            'id' => $row['order_id'],
            'total' => $row['total'],
        ];
    }
}
// Total: 1 query!
```

Solution 2 — Eager loading with IN (two queries):

```php
<?php
// Query 1: Get users
$users = $pdo->query('SELECT * FROM users LIMIT 100')->fetchAll();
$userIds = array_column($users, 'id');

// Query 2: Get all orders for those users
$placeholders = implode(',', array_fill(0, count($userIds), '?'));
$stmt = $pdo->prepare(
    "SELECT * FROM orders WHERE user_id IN ($placeholders)"
);
$stmt->execute($userIds);
$allOrders = $stmt->fetchAll();

// Group orders by user_id
$ordersByUser = [];
foreach ($allOrders as $order) {
    $ordersByUser[$order['user_id']][] = $order;
}

// Attach to users
foreach ($users as &$user) {
    $user['orders'] = $ordersByUser[$user['id']] ?? [];
}
// Total: 2 queries (regardless of user count)
```

A reusable eager loader:

```php
<?php
class EagerLoader {
    public function __construct(private PDO $pdo) {}
    
    public function loadRelation(
        array &$items,
        string $relation,
        string $table,
        string $foreignKey,
        string $localKey = 'id'
    ): void {
        $ids = array_unique(array_column($items, $localKey));
        if (empty($ids)) {
            return;
        }
        
        $placeholders = implode(',', array_fill(0, count($ids), '?'));
        $stmt = $this->pdo->prepare(
            "SELECT * FROM $table WHERE $foreignKey IN ($placeholders)"
        );
        $stmt->execute(array_values($ids));
        $related = $stmt->fetchAll();
        
        $grouped = [];
        foreach ($related as $row) {
            $grouped[$row[$foreignKey]][] = $row;
        }
        
        foreach ($items as &$item) {
            $item[$relation] = $grouped[$item[$localKey]] ?? [];
        }
    }
}
```

Choose JOINs when you need a single query and the data relationship is simple. Choose IN-based eager loading when you need to load multiple independent relationships or when the JOIN would produce too many duplicate rows.

## Common Pitfalls

1. **Not noticing N+1 queries** — They hide inside loops and look innocent. Use a query logger or debug toolbar to count queries per request.
2. **Using JOINs for everything** — JOINing many tables with one-to-many relationships multiplies rows, sending redundant data. IN-based loading is often more efficient for multiple relationships.

## Best Practices

1. **Count queries per request during development** — Anything over ~10 queries per page deserves investigation.
2. **Use eager loading for any relationship accessed in a loop** — If you touch related data inside a foreach, you have an N+1 problem.

## Summary

- The N+1 problem turns O(1) queries into O(N) queries by loading related data one item at a time.
- JOINs solve it with a single query; IN-based eager loading solves it with two queries.
- A reusable EagerLoader class can batch-load any relationship.
- Always count queries per request during development to catch N+1 issues early.

## Code Examples

**Eager loading multiple relationships efficiently**

```php
<?php
declare(strict_types=1);

$loader = new EagerLoader($pdo);
$users = $pdo->query('SELECT * FROM users LIMIT 50')->fetchAll();

// Load multiple relationships: 3 queries total
$loader->loadRelation($users, 'orders', 'orders', 'user_id');
$loader->loadRelation($users, 'posts', 'posts', 'author_id');

foreach ($users as $user) {
    $orderCount = count($user['orders']);
    $postCount = count($user['posts']);
    echo "{$user['name']}: {$orderCount} orders, {$postCount} posts\n";
}
?>
```


## Resources

- [N+1 Query Problem Explained](https://stackoverflow.com/questions/97197/what-is-the-n1-selects-problem-in-orm) — Classic explanation of the N+1 problem

---

> 📘 *This lesson is part of the [PHP & Relational Databases](https://stanza.dev/courses/php-databases) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*