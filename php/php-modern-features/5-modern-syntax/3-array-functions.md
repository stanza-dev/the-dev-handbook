---
source_course: "php-modern-features"
source_lesson: "php-modern-features-new-array-functions"
---

# New Array Functions

## Introduction
PHP 8.4 introduced four powerful array functions — `array_find()`, `array_find_key()`, `array_any()`, and `array_all()` — that eliminate the need for custom `foreach` loops for common search and validation patterns. PHP 8.5 adds `array_first()` and `array_last()` for simple element access.

## Key Concepts
- **`array_find()`**: Returns the first element matching a callback condition, or `null`.
- **`array_any()` / `array_all()`**: Boolean checks — do any/all elements satisfy a condition?
- **`array_first()` / `array_last()` (PHP 8.5)**: Return the first or last value of an array without needing `reset()` or `end()`.

## Real World Context
JavaScript developers have had `find()`, `some()`, and `every()` for years. PHP developers wrote the same `foreach` loops thousands of times. These native functions bring PHP's array API up to parity with other languages, improving readability and reducing bugs.

## Deep Dive

### array_find()

Finds the first element matching a condition:

```php
<?php
$users = [
    ['id' => 1, 'name' => 'Alice', 'admin' => false],
    ['id' => 2, 'name' => 'Bob', 'admin' => true],
    ['id' => 3, 'name' => 'Charlie', 'admin' => false],
];

$admin = array_find($users, fn($user) => $user['admin'] === true);
// ['id' => 2, 'name' => 'Bob', 'admin' => true]

$notFound = array_find($users, fn($user) => $user['id'] === 999);
// null
```

It returns the element itself (not the key), or `null` if no match is found.

### array_find_key()

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

Useful when you need the key for further operations like `unset()` or array access.

### array_any()

Checks if ANY element matches:

```php
<?php
$numbers = [1, 2, 3, 4, 5];

$hasEven = array_any($numbers, fn($n) => $n % 2 === 0);
// true

$hasNegative = array_any($numbers, fn($n) => $n < 0);
// false
```

Short-circuits on the first match, so it is efficient for large arrays.

### array_all()

Checks if ALL elements match:

```php
<?php
$scores = [85, 90, 78, 92, 88];

$allPassed = array_all($scores, fn($score) => $score >= 60);
// true

$allExcellent = array_all($scores, fn($score) => $score >= 90);
// false
```

Short-circuits on the first non-match.

### array_first() and array_last() (PHP 8.5)

Get the first or last value without side effects:

```php
<?php
$items = ['apple', 'banana', 'cherry'];

$first = array_first($items);  // 'apple'
$last = array_last($items);    // 'cherry'

$empty = [];
$result = array_first($empty);  // null
```

Before PHP 8.5, getting the first element required `reset($array)` (which mutates the internal pointer) or `$array[array_key_first($array)]` (verbose). `array_first()` and `array_last()` are simple, non-mutating, and return `null` for empty arrays.

### Comparison with Traditional Approach

These functions replace boilerplate loops:

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

// PHP 8.4+ - native function
$found = array_find($array, $callback);
```

The native functions are more concise and communicate intent clearly.

## Common Pitfalls
1. **Confusing `array_find()` with `array_search()`** — `array_search()` searches by value and returns a key. `array_find()` searches by callback and returns the element. They solve different problems.
2. **Expecting `array_any()` to return the matching element** — `array_any()` returns `true` or `false`, not the element. Use `array_find()` if you need the actual element.

## Best Practices
1. **Use `array_any()` for validation** — `array_any($responses, fn($r) => $r['status'] >= 400)` is cleaner than a foreach loop with a flag variable.
2. **Use `array_first()` instead of `reset()`** — `reset()` has the side effect of modifying the array's internal pointer. `array_first()` is pure and intention-revealing.

## Summary
- `array_find()` returns the first matching element; `array_find_key()` returns its key.
- `array_any()` and `array_all()` provide boolean validation over arrays.
- PHP 8.5 adds `array_first()` and `array_last()` for non-mutating element access.
- These functions replace common foreach-with-flag patterns.
- They short-circuit for efficiency on large arrays.

## Code Examples

**Practical order processing using array_find, array_any, array_all, array_first, and array_last**

```php
<?php
declare(strict_types=1);

$orders = [
    ['id' => 1, 'status' => 'shipped', 'total' => 99.99],
    ['id' => 2, 'status' => 'pending', 'total' => 149.50],
    ['id' => 3, 'status' => 'delivered', 'total' => 75.00],
];

// Find first pending order
$pendingOrder = array_find($orders, fn($o) => $o['status'] === 'pending');
// ['id' => 2, 'status' => 'pending', 'total' => 149.50]

// Check if any order exceeds limit
$hasLargeOrder = array_any($orders, fn($o) => $o['total'] > 100);
// true

// Verify all orders are under budget
$allUnderBudget = array_all($orders, fn($o) => $o['total'] < 200);
// true

// PHP 8.5: Get first and last orders
$firstOrder = array_first($orders);  // ['id' => 1, ...]
$lastOrder = array_last($orders);    // ['id' => 3, ...]
?>
```


## Resources

- [Array Functions](https://www.php.net/manual/en/ref.array.php) — Complete PHP array functions reference

---

> 📘 *This lesson is part of the [Modern PHP 8.x: Latest Language Features](https://stanza.dev/courses/php-modern-features) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*