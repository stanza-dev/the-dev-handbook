---
source_course: "php-essentials"
source_lesson: "php-essentials-arrays-fundamentals"
---

# Array Fundamentals

## Introduction

Arrays are the most versatile and commonly used data structure in PHP. Whether you are building a shopping cart, storing user records, or processing API data, arrays are at the heart of nearly every PHP application. In this lesson, you will learn how to create, access, and manipulate arrays in all their forms.

## Key Concepts

- **Indexed array**: An array where elements are accessed by numeric positions starting at 0.
- **Associative array**: An array where elements are accessed by named string keys.
- **Multi-dimensional array**: An array that contains other arrays as its elements.
- **Array destructuring**: A syntax for extracting values from arrays into separate variables.
- **Spread operator (`...`)**: Unpacks array elements into a new array literal.

## Real World Context

Every real PHP application deals with collections of data. A list of products from a database, form field values, configuration settings, JSON API responses — all of these are represented as arrays. Mastering arrays is not optional; it is a prerequisite for working with any PHP framework or library.

## Deep Dive

### Creating Arrays

PHP offers two syntaxes for creating arrays. The short bracket syntax is the modern standard:

```php
<?php
// Short syntax (recommended)
$fruits = ['apple', 'banana', 'cherry'];

// Legacy syntax
$colors = array('red', 'green', 'blue');

// Empty array
$empty = [];
```

The short syntax `[]` is preferred in all modern PHP codebases. The `array()` constructor still works but is considered legacy.

### Indexed Arrays

Indexed arrays use numeric keys starting at 0. You can append elements with the `[]` operator:

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

Notice that `$fruits[]` without a key appends to the end. This is the idiomatic way to push a single element.

### Associative Arrays

Associative arrays use string keys with the `=>` operator. They are ideal for representing structured data like a user record:

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

The trailing comma after the last element is a best practice that makes diffs cleaner when adding new entries.

### Multi-dimensional Arrays

Arrays can contain other arrays, creating nested structures:

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

Access nested values by chaining bracket notation. The first bracket selects the outer array element, the second selects within it.

### Array Destructuring (PHP 7.1+)

Destructuring lets you extract values from an array into individual variables in a single statement:

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

This is especially useful when working with database rows or function return values.

### Spread Operator

The spread operator `...` unpacks array elements into a new array:

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

Note that associative array spreading requires PHP 8.1 or later. For indexed arrays, it works from PHP 7.4 onward.

## Common Pitfalls

1. **Accessing undefined keys without a fallback** — Using `$arr['key']` when the key may not exist triggers a warning. Always use the null coalescing operator: `$arr['key'] ?? 'default'`.
2. **Confusing indexed and associative behavior** — PHP arrays are actually ordered maps. Mixing numeric and string keys in the same array can lead to surprising reindexing behavior. Keep arrays consistently indexed or consistently associative.
3. **Forgetting that arrays are copied on assignment** — `$b = $a` creates a full copy. Modifying `$b` does not change `$a`. Use references (`&`) only when you explicitly need shared mutation.

## Best Practices

1. **Use short array syntax** — Always prefer `[]` over `array()`. It is shorter, cleaner, and the community standard since PHP 5.4.
2. **Add trailing commas** — Place a comma after the last element in multi-line arrays. This produces cleaner version control diffs and prevents syntax errors when reordering.
3. **Use descriptive string keys** — For associative arrays, choose self-documenting keys like `'email'` instead of abbreviations like `'e'`.

## Summary

- PHP arrays are ordered maps that can serve as lists, dictionaries, stacks, and queues.
- Indexed arrays use numeric keys starting at 0; associative arrays use string keys with `=>`.
- Multi-dimensional arrays nest arrays within arrays, accessed by chaining brackets.
- Destructuring (`[$a, $b] = $arr`) extracts values into variables in one statement.
- The spread operator (`...`) merges arrays inline without calling `array_merge()`.

## Code Examples

**Building a shopping cart with indexed and associative arrays combined**

```php
<?php
// Building a shopping cart with arrays
$cart = [];

// Add items as associative arrays
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

// Calculate total using a loop
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