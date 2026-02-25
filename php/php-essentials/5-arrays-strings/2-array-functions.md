---
source_course: "php-essentials"
source_lesson: "php-essentials-array-functions"
---

# Essential Array Functions

## Introduction

PHP ships with over 80 built-in array functions, making it one of the richest standard libraries for array manipulation. Knowing the right function for each task saves you from writing error-prone loops and keeps your code expressive. This lesson covers the functions you will use most frequently.

## Key Concepts

- **Counting**: `count()` returns the number of elements in an array.
- **Searching**: `in_array()`, `array_search()`, and `array_key_exists()` locate values and keys.
- **Transforming**: `array_map()`, `array_filter()`, and `array_reduce()` process arrays functionally.
- **Sorting**: `sort()`, `usort()`, `asort()`, and `ksort()` reorder elements in place.
- **Merging**: `array_merge()` and the spread operator combine arrays.

## Real World Context

In a real application, you rarely write raw loops over arrays. Instead, you pipe data through transformation functions: filter out inactive users, map database rows into DTOs, reduce order items into a total price. These array functions are the PHP equivalent of SQL's WHERE, SELECT, and SUM — knowing them well makes your data processing code concise and readable.

## Deep Dive

### Counting and Checking

Use `count()` to get the array length, and `in_array()` or `array_key_exists()` to check membership:

```php
<?php
$arr = [1, 2, 3, 4, 5];

count($arr);           // 5
sizeof($arr);          // 5 (alias)

in_array(3, $arr);     // true - value exists
array_key_exists('name', $user); // check key
```

`in_array()` checks values while `array_key_exists()` checks keys. Use strict mode (`true` as the third argument to `in_array`) to avoid type coercion surprises.

### Adding and Removing

PHP provides stack-like and queue-like operations:

```php
<?php
$stack = [1, 2, 3];

array_push($stack, 4, 5);  // [1,2,3,4,5] - add to end
array_pop($stack);          // returns 5, stack is [1,2,3,4]

array_unshift($stack, 0);   // [0,1,2,3,4] - add to start
array_shift($stack);        // returns 0, stack is [1,2,3,4]

unset($stack[1]);           // Remove by key
```

Note that `unset()` does not reindex the array. Use `array_values()` afterward if you need consecutive integer keys.

### Searching

`array_search()` returns the key of a found value, and PHP 8.4 introduces the `array_find()` family:

```php
<?php
$fruits = ['apple', 'banana', 'cherry'];

array_search('banana', $fruits);  // 1 (index)

// PHP 8.4+: New array_find functions
$users = [
    ['id' => 1, 'name' => 'Alice'],
    ['id' => 2, 'name' => 'Bob'],
];

$bob = array_find($users, fn($u) => $u['name'] === 'Bob');
// ['id' => 2, 'name' => 'Bob']
```

`array_search()` returns `false` when not found — always compare with `===` to distinguish from index `0`.

### Transforming

The functional trio — map, filter, reduce — lets you process arrays without writing loops:

```php
<?php
$numbers = [1, 2, 3, 4, 5];

// Map - transform each element
$doubled = array_map(fn($n) => $n * 2, $numbers);
// [2, 4, 6, 8, 10]

// Filter - keep matching elements
$evens = array_filter($numbers, fn($n) => $n % 2 === 0);
// [2, 4]

// Reduce - collapse to single value
$sum = array_reduce($numbers, fn($carry, $n) => $carry + $n, 0);
// 15
```

`array_filter()` preserves keys, so you may need `array_values()` to reindex the result.

### Sorting

Sorting functions modify the array in place and return a boolean:

```php
<?php
$arr = [3, 1, 4, 1, 5];

sort($arr);         // [1, 1, 3, 4, 5] - ascending
rsort($arr);        // [5, 4, 3, 1, 1] - descending

// Preserve keys
asort($arr);        // Sort by value, keep keys
ksort($arr);        // Sort by key

// Custom sorting
usort($arr, fn($a, $b) => $b - $a);  // Descending
```

Use `usort()` with the spaceship operator (`<=>`) for custom comparisons: `fn($a, $b) => $a <=> $b`.

### Merging and Combining

`array_merge()` combines arrays, and `array_combine()` creates key-value pairs from two arrays:

```php
<?php
$a = ['x' => 1, 'y' => 2];
$b = ['y' => 3, 'z' => 4];

array_merge($a, $b);      // ['x'=>1, 'y'=>3, 'z'=>4]
array_merge_recursive($a, $b);  // Keeps both values

$keys = ['a', 'b', 'c'];
$values = [1, 2, 3];
array_combine($keys, $values);  // ['a'=>1, 'b'=>2, 'c'=>3]
```

With `array_merge()`, later values overwrite earlier ones for duplicate string keys. Numeric keys are always reindexed.

### Extracting

`array_slice()` returns a portion of the array without modifying the original:

```php
<?php
$arr = [1, 2, 3, 4, 5];

array_slice($arr, 2);      // [3, 4, 5] - from index 2
array_slice($arr, 1, 2);   // [2, 3] - 2 elements from index 1

array_keys($arr);          // [0, 1, 2, 3, 4]
array_values($arr);        // Re-index array
```

## Common Pitfalls

1. **Forgetting that `array_filter()` preserves keys** — After filtering, the array keys may have gaps (e.g., `[0 => 'a', 2 => 'c']`). Wrap with `array_values()` when you need consecutive indices.
2. **Using `in_array()` without strict mode** — `in_array(0, ['a', 'b'])` returns `true` due to type juggling. Always pass `true` as the third argument: `in_array(0, $arr, true)`.
3. **Sorting functions return booleans, not arrays** — `sort($arr)` modifies `$arr` in place and returns `true`. Writing `$sorted = sort($arr)` gives you `true`, not the sorted array.

## Best Practices

1. **Prefer functional transformations over loops** — `array_map()`, `array_filter()`, and `array_reduce()` express intent more clearly than `foreach` with manual array building.
2. **Use arrow functions for short callbacks** — `fn($x) => $x * 2` is cleaner than `function($x) { return $x * 2; }` for simple operations.
3. **Chain operations in a readable pipeline** — When applying multiple transformations, consider assigning intermediate results to descriptively named variables for clarity.

## Summary

- `count()` gives array length; `in_array()` and `array_key_exists()` check membership.
- `array_push()` / `array_pop()` work like stacks; `array_unshift()` / `array_shift()` work like queues.
- `array_map()`, `array_filter()`, and `array_reduce()` transform arrays functionally without loops.
- Sorting functions (`sort`, `usort`, `asort`, `ksort`) modify arrays in place.
- `array_merge()` combines arrays; `array_slice()` extracts portions without mutation.

## Code Examples

**Data processing pipeline using array_filter, array_map, and array_reduce to calculate total revenue from completed orders**

```php
<?php
// Data processing pipeline: filter, map, reduce
$orders = [
    ['id' => 1, 'total' => 150, 'status' => 'completed'],
    ['id' => 2, 'total' => 75, 'status' => 'pending'],
    ['id' => 3, 'total' => 200, 'status' => 'completed'],
    ['id' => 4, 'total' => 50, 'status' => 'cancelled'],
    ['id' => 5, 'total' => 300, 'status' => 'completed'],
];

// Pipeline: Filter completed -> extract totals -> sum them
$completedRevenue = array_reduce(
    array_map(
        fn($o) => $o['total'],
        array_filter(
            $orders,
            fn($o) => $o['status'] === 'completed'
        )
    ),
    fn($sum, $total) => $sum + $total,
    0
);

echo "Completed Revenue: \$$completedRevenue";
// Completed Revenue: $650
?>
```


## Resources

- [Array Functions](https://www.php.net/manual/en/ref.array.php) — Complete reference of PHP array functions

---

> 📘 *This lesson is part of the [PHP Essentials](https://stanza.dev/courses/php-essentials) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*