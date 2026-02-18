---
source_course: "php-essentials"
source_lesson: "php-essentials-arrays-fundamentals"
---

# Array Fundamentals

Arrays in PHP are ordered maps that can hold multiple values. They're one of the most versatile data structures.

## Creating Arrays

```php
<?php
// Short syntax (recommended)
$fruits = ['apple', 'banana', 'cherry'];

// Legacy syntax
$colors = array('red', 'green', 'blue');

// Empty array
$empty = [];
```

## Indexed Arrays

Numerically indexed, starting at 0:

```php
<?php
$fruits = ['apple', 'banana', 'cherry'];

echo $fruits[0];  // apple
echo $fruits[1];  // banana
echo $fruits[2];  // cherry

// Add to the end
$fruits[] = 'date';
// Now: ['apple', 'banana', 'cherry', 'date']
```

## Associative Arrays

Key-value pairs:

```php
<?php
$user = [
    'name' => 'John Doe',
    'email' => 'john@example.com',
    'age' => 30,
    'active' => true,
];

echo $user['name'];   // John Doe
echo $user['email'];  // john@example.com

// Add/update
$user['phone'] = '555-1234';
$user['age'] = 31;  // Update existing
```

## Multi-dimensional Arrays

```php
<?php
$matrix = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9],
];

echo $matrix[1][2];  // 6

$users = [
    ['name' => 'Alice', 'age' => 25],
    ['name' => 'Bob', 'age' => 30],
];

echo $users[0]['name'];  // Alice
```

## Array Destructuring (PHP 7.1+)

```php
<?php
$coordinates = [10, 20, 30];
[$x, $y, $z] = $coordinates;
echo "$x, $y, $z";  // 10, 20, 30

// Skip values
[, , $z] = $coordinates;

// With keys
$user = ['name' => 'John', 'email' => 'john@test.com'];
['name' => $name, 'email' => $email] = $user;
```

## Spread Operator

```php
<?php
$first = [1, 2, 3];
$second = [4, 5, 6];

$combined = [...$first, ...$second];
// [1, 2, 3, 4, 5, 6]

// With associative arrays (PHP 8.1+)
$defaults = ['color' => 'blue', 'size' => 'M'];
$custom = ['color' => 'red'];
$merged = [...$defaults, ...$custom];
// ['color' => 'red', 'size' => 'M']
```

## Code Examples

**Building a shopping cart with arrays**

```php
<?php
// Building a shopping cart
$cart = [];

// Add items
$cart[] = [
    'id' => 101,
    'name' => 'Laptop',
    'price' => 999.99,
    'quantity' => 1
];

$cart[] = [
    'id' => 205,
    'name' => 'Mouse',
    'price' => 29.99,
    'quantity' => 2
];

// Calculate total
$total = 0;
foreach ($cart as $item) {
    $total += $item['price'] * $item['quantity'];
}

echo "Cart Total: \$" . number_format($total, 2);
// Cart Total: $1,059.97
?>
```


## Resources

- [PHP Arrays](https://www.php.net/manual/en/language.types.array.php) — Complete guide to PHP arrays

---

> 📘 *This lesson is part of the [PHP Essentials](https://stanza.dev/courses/php-essentials) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*