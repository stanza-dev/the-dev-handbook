---
source_course: "php-essentials"
source_lesson: "php-essentials-loops"
---

# Loops in PHP

## Introduction
Loops let you execute code repeatedly, which is essential for processing collections, generating output, and running tasks until a condition is met. This lesson covers all four PHP loop types and the control statements (`break`, `continue`) that manage loop flow.

## Key Concepts
- **`for` Loop**: Best when you know the exact number of iterations ahead of time.
- **`while` Loop**: Continues as long as a condition remains true. The condition is checked before each iteration.
- **`do-while` Loop**: Like `while`, but the body executes at least once because the condition is checked after.
- **`foreach` Loop**: Designed specifically for iterating over arrays and objects. The most commonly used loop in PHP.
- **`break` and `continue`**: Control statements that exit a loop or skip to the next iteration.

## Real World Context
Processing database results, iterating over form inputs, rendering lists of products, paginating API responses — loops are everywhere in web development. The `foreach` loop is by far the most common in PHP because PHP applications work heavily with arrays (query results, JSON data, configuration arrays). Understanding `break` and `continue` helps you write efficient loops that skip unnecessary work.

## Deep Dive

### `for` Loop

Best when you know the number of iterations upfront:

```php
<?php
for ($i = 0; $i < 5; $i++) {
    echo "Iteration: $i\n";
}
// Outputs: 0, 1, 2, 3, 4
```

The syntax is `for (init; condition; increment)`. The init runs once, the condition is checked before each iteration, and the increment runs after each iteration.

### `while` Loop

Continue while a condition is true:

```php
<?php
$count = 0;

while ($count < 3) {
    echo "Count: $count\n";
    $count++;
}
// Outputs: 0, 1, 2
```

If the condition is false initially, the body never executes.

### `do-while` Loop

Executes the body at least once, then checks the condition:

```php
<?php
$x = 10;

do {
    echo "x is: $x\n";
    $x++;
} while ($x < 5);
// Outputs: "x is: 10" (runs once even though 10 > 5)
```

This is useful for menus, retry logic, or any scenario where you need at least one execution.

### `foreach` Loop

The go-to loop for arrays and objects:

```php
<?php
$fruits = ['apple', 'banana', 'cherry'];

// Value only
foreach ($fruits as $fruit) {
    echo "$fruit\n";
}

// Key and value
foreach ($fruits as $index => $fruit) {
    echo "$index: $fruit\n";
}

// Associative array
$user = ['name' => 'John', 'age' => 30];
foreach ($user as $key => $value) {
    echo "$key = $value\n";
}
```

The `foreach` loop handles both indexed and associative arrays seamlessly.

### Loop Control: `break` and `continue`

`break` exits the loop entirely:

```php
<?php
for ($i = 0; $i < 10; $i++) {
    if ($i === 5) {
        break;  // Stop at 5
    }
    echo $i;
}
// Outputs: 01234
```

`continue` skips to the next iteration:

```php
<?php
for ($i = 0; $i < 5; $i++) {
    if ($i === 2) {
        continue;  // Skip 2
    }
    echo $i;
}
// Outputs: 0134
```

### Breaking Nested Loops

Pass a number to `break` to exit multiple levels:

```php
<?php
for ($i = 0; $i < 3; $i++) {
    for ($j = 0; $j < 3; $j++) {
        if ($j === 1) {
            break 2;  // Break out of both loops
        }
    }
}
```

The number after `break` specifies how many nested loops to exit.

### Modifying Arrays in Foreach

Use a reference (`&`) to modify array elements in place:

```php
<?php
$numbers = [1, 2, 3];

foreach ($numbers as &$num) {
    $num *= 2;
}
unset($num);  // Important! Break the reference

print_r($numbers);  // [2, 4, 6]
```

Always `unset()` the reference variable after the loop to prevent unexpected behavior in later code.

## Common Pitfalls
1. **Forgetting to `unset()` after a reference foreach** — The `$num` variable remains a reference to the last array element. Reusing it later will silently modify the array.
2. **Infinite loops** — Forgetting to increment the counter in a `while` loop creates an infinite loop. Always ensure the condition will eventually become false.
3. **Modifying an array while iterating** — Adding or removing elements during a `foreach` leads to unpredictable results. Collect changes in a separate array and apply them after the loop.

## Best Practices
1. **Prefer `foreach` for arrays** — It is cleaner, safer, and more idiomatic than using `for` with manual index management.
2. **Extract complex loop bodies into functions** — If your loop body exceeds 10-15 lines, move the logic into a separate function for readability.
3. **Use `break` with early returns** — When searching for a value, `break` as soon as you find it. Do not iterate through the rest of the array unnecessarily.

## Summary
- `for` loops are for known iteration counts; `while` and `do-while` are for condition-based loops.
- `foreach` is the most common loop in PHP, designed for arrays and objects.
- `break` exits a loop; `continue` skips to the next iteration.
- `break 2` exits two nested loops at once.
- Always `unset()` reference variables after a `foreach` with `&`.

## Code Examples

**Processing orders with foreach and continue — skips non-pending orders efficiently**

```php
<?php
// Real example: Processing a list of orders
$orders = [
    ['id' => 1, 'total' => 99.99, 'status' => 'pending'],
    ['id' => 2, 'total' => 149.50, 'status' => 'shipped'],
    ['id' => 3, 'total' => 25.00, 'status' => 'pending'],
    ['id' => 4, 'total' => 200.00, 'status' => 'delivered'],
];

$pendingTotal = 0;
$pendingCount = 0;

foreach ($orders as $order) {
    if ($order['status'] !== 'pending') {
        continue;  // Skip non-pending orders
    }

    $pendingTotal += $order['total'];
    $pendingCount++;
}

echo "Pending orders: $pendingCount\n";
echo "Pending total: \$$pendingTotal\n";
// Pending orders: 2
// Pending total: $124.99
?>
```


## Resources

- [PHP Loops](https://www.php.net/manual/en/language.control-structures.php) — Official documentation for all PHP loop types

---

> 📘 *This lesson is part of the [PHP Essentials](https://stanza.dev/courses/php-essentials) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*