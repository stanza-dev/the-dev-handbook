---
source_course: "php-modern-features"
source_lesson: "php-modern-features-new-array-functions"
---

# New Array Functions in PHP 8.4

PHP 8.4 introduces powerful array functions that reduce boilerplate and improve readability.

## array_find()

Finds the first element matching a condition:

```php
<?php
$users = [
    ['id' => 1, 'name' => 'Alice', 'admin' => false],
    ['id' => 2, 'name' => 'Bob', 'admin' => true],
    ['id' => 3, 'name' => 'Charlie', 'admin' => false],
];

// Find first admin
$admin = array_find($users, fn($user) => $user['admin'] === true);
// ['id' => 2, 'name' => 'Bob', 'admin' => true]

// Returns null if not found
$notFound = array_find($users, fn($user) => $user['id'] === 999);
// null
```

## array_find_key()

Returns the key of the first matching element:

```php
<?php
$products = [
    'widget' => ['price' => 10],
    'gadget' => ['price' => 25],
    'gizmo' => ['price' => 15],
];

$key = array_find_key($products, fn($p) => $p['price'] > 20);
// 'gadget'
```

## array_any()

Checks if ANY element matches a condition:

```php
<?php
$numbers = [1, 2, 3, 4, 5];

$hasEven = array_any($numbers, fn($n) => $n % 2 === 0);
// true

$hasNegative = array_any($numbers, fn($n) => $n < 0);
// false

// Useful for validation
$hasErrors = array_any($responses, fn($r) => $r['status'] >= 400);
if ($hasErrors) {
    // Handle error case
}
```

## array_all()

Checks if ALL elements match a condition:

```php
<?php
$scores = [85, 90, 78, 92, 88];

$allPassed = array_all($scores, fn($score) => $score >= 60);
// true

$allExcellent = array_all($scores, fn($score) => $score >= 90);
// false

// Validate all items
$allValid = array_all($items, fn($item) => $item['quantity'] > 0);
```

## Comparison with Traditional Approach

```php
<?php
// Before PHP 8.4
function findFirst(array $array, callable $callback): mixed {
    foreach ($array as $item) {
        if ($callback($item)) {
            return $item;
        }
    }
    return null;
}

function any(array $array, callable $callback): bool {
    foreach ($array as $item) {
        if ($callback($item)) {
            return true;
        }
    }
    return false;
}

// PHP 8.4 - native functions
$found = array_find($array, $callback);
$hasMatch = array_any($array, $callback);
```

## Practical Examples

```php
<?php
$orders = [
    ['id' => 1, 'status' => 'shipped', 'total' => 99.99],
    ['id' => 2, 'status' => 'pending', 'total' => 149.50],
    ['id' => 3, 'status' => 'delivered', 'total' => 75.00],
];

// Find pending order
$pendingOrder = array_find($orders, fn($o) => $o['status'] === 'pending');

// Check if any order exceeds limit
$hasLargeOrder = array_any($orders, fn($o) => $o['total'] > 100);

// Verify all orders are under budget
$allUnderBudget = array_all($orders, fn($o) => $o['total'] < 200);
```

## Resources

- [Array Functions](https://www.php.net/manual/en/ref.array.php) — Complete PHP array functions reference

---

> 📘 *This lesson is part of the [Modern PHP 8.x: Latest Language Features](https://stanza.dev/courses/php-modern-features) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*