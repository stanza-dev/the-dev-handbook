---
source_course: "php-essentials"
source_lesson: "php-essentials-array-functions"
---

# Essential Array Functions

PHP provides dozens of built-in functions for array manipulation.

## Counting & Checking

```php
<?php
$arr = [1, 2, 3, 4, 5];

count($arr);           // 5
sizeof($arr);          // 5 (alias)

in_array(3, $arr);     // true - value exists
array_key_exists('name', $user); // check key
```

## Adding & Removing

```php
<?php
$stack = [1, 2, 3];

array_push($stack, 4, 5);  // [1,2,3,4,5] - add to end
array_pop($stack);          // returns 5, stack is [1,2,3,4]

array_unshift($stack, 0);   // [0,1,2,3,4] - add to start
array_shift($stack);        // returns 0, stack is [1,2,3,4]

unset($stack[1]);           // Remove by key
```

## Searching

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

## Transforming

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

## Sorting

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

## Merging & Combining

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

## Extracting

```php
<?php
$arr = [1, 2, 3, 4, 5];

array_slice($arr, 2);      // [3, 4, 5] - from index 2
array_slice($arr, 1, 2);   // [2, 3] - 2 elements from index 1

array_keys($arr);          // [0, 1, 2, 3, 4]
array_values($arr);        // Re-index array
```

## Code Examples

**Data processing pipeline with array functions**

```php
<?php
// Data processing pipeline
$orders = [
    ['id' => 1, 'total' => 150, 'status' => 'completed'],
    ['id' => 2, 'total' => 75, 'status' => 'pending'],
    ['id' => 3, 'total' => 200, 'status' => 'completed'],
    ['id' => 4, 'total' => 50, 'status' => 'cancelled'],
    ['id' => 5, 'total' => 300, 'status' => 'completed'],
];

// Pipeline: Filter -> Map -> Reduce
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