---
source_course: "php-essentials"
source_lesson: "php-essentials-loops"
---

# Loops in PHP

Loops let you execute code repeatedly. PHP offers several loop types for different scenarios.

## `for` Loop

Best when you know the number of iterations:

```php
<?php
for ($i = 0; $i < 5; $i++) {
    echo "Iteration: $i\n";
}
// Outputs: 0, 1, 2, 3, 4
```

Anatomy: `for (init; condition; increment)`

## `while` Loop

Continue while condition is true:

```php
<?php
$count = 0;

while ($count < 3) {
    echo "Count: $count\n";
    $count++;
}
// Outputs: 0, 1, 2
```

## `do-while` Loop

Executes at least once:

```php
<?php
$x = 10;

do {
    echo "x is: $x\n";
    $x++;
} while ($x < 5);
// Outputs: "x is: 10" (runs once even though 10 > 5)
```

## `foreach` Loop

Designed for arrays and objects:

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

## Loop Control

### `break` - Exit the loop entirely

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

### `continue` - Skip to next iteration

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

### Breaking nested loops

```php
<?php
for ($i = 0; $i < 3; $i++) {
    for ($j = 0; $j < 3; $j++) {
        if ($j === 1) {
            break 2;  // Break out of 2 loops
        }
    }
}
```

## Modifying Arrays in Foreach

```php
<?php
$numbers = [1, 2, 3];

// Use reference (&) to modify
foreach ($numbers as &$num) {
    $num *= 2;
}
unset($num);  // Important! Break the reference

print_r($numbers);  // [2, 4, 6]
```

## Code Examples

**Processing orders with foreach and continue**

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
?>
```


## Resources

- [PHP Loops](https://www.php.net/manual/en/language.control-structures.php) — Official documentation for all PHP loop types

---

> 📘 *This lesson is part of the [PHP Essentials](https://stanza.dev/courses/php-essentials) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*