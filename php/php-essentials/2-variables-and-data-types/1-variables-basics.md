---
source_course: "php-essentials"
source_lesson: "php-essentials-variables-basics"
---

# Understanding Variables

## Introduction
Variables are the building blocks of every PHP program. They let you store, retrieve, and manipulate data as your script runs. This lesson covers PHP's variable syntax, naming rules, assignment patterns, and the essential functions for checking and destroying variables.

## Key Concepts
- **Variable (`$name`)**: A named container for data. In PHP, all variables start with a dollar sign (`$`).
- **Assignment Operator (`=`)**: Stores a value in a variable. The right side is evaluated first, then the result is placed in the variable on the left.
- **Dynamic Typing**: PHP variables can hold any type of value and can change type during execution.

## Real World Context
Every server-side script uses variables to hold user input, database results, configuration values, and computed output. Understanding how PHP variables work — including their naming rules and checking functions — prevents the subtle bugs that come from undefined variables and unexpected null values.

## Deep Dive

### Variable Syntax

In PHP, all variables start with a dollar sign (`$`), followed by the name:

```php
<?php
$name = "John";      // String
$age = 25;           // Integer
$price = 19.99;      // Float
$isActive = true;    // Boolean
```

Each variable stores a different type of data, and PHP determines the type automatically.

### Variable Naming Rules

PHP has strict rules for variable names:

1. Must start with a letter or underscore (`_`)
2. Can contain letters, numbers, and underscores
3. Case-sensitive (`$name` is not the same as `$Name`)
4. Cannot start with a number

Here are examples of valid and invalid names:

```php
<?php
// Valid variable names
$userName = "Alice";
$user_name = "Bob";
$_private = "secret";
$item1 = "first";

// Invalid variable names
// $1item = "wrong";     // Cannot start with number
// $user-name = "wrong"; // Hyphens not allowed
```

The convention in PHP is to use camelCase (`$userName`) or snake_case (`$user_name`). Pick one style and be consistent.

### Variable Assignment

Use the assignment operator (`=`) to give variables values:

```php
<?php
$message = "Hello";        // Assign a value
$message = "Goodbye";      // Reassign (overwrite)

$a = $b = $c = 10;         // Multiple assignment
```

Multiple assignment evaluates right-to-left: `$c` gets 10 first, then `$b`, then `$a`.

### Variable Variables

PHP allows variable variable names, where one variable's value becomes another variable's name:

```php
<?php
$varName = "greeting";
$$varName = "Hello!";  // Creates $greeting

echo $greeting;  // Outputs: Hello!
```

This is a powerful but dangerous feature. Use it sparingly, as it makes code harder to read and debug.

### Checking Variables

PHP provides built-in functions to inspect and manage variables:

```php
<?php
$name = "Alice";

isset($name);      // true - variable exists and not null
isset($unknown);   // false - variable doesn't exist

empty("");         // true - empty string is "empty"
empty(0);          // true - zero is considered "empty"
empty("hello");    // false - has content

unset($name);      // Destroy the variable
isset($name);      // false - no longer exists
```

`isset()` checks that a variable exists and is not null. `empty()` checks for "empty" values including `""`, `0`, `null`, `false`, and empty arrays.

## Common Pitfalls
1. **Forgetting the dollar sign** — Writing `name = "Alice"` instead of `$name = "Alice"` causes a parse error. Every variable in PHP requires the `$` prefix.
2. **Case sensitivity surprises** — `$user` and `$User` are two different variables. A typo in casing leads to undefined variable warnings.
3. **Relying on `empty()` for zero checks** — `empty(0)` returns `true`, which can cause bugs when zero is a valid value. Use `isset()` and explicit comparisons instead.

## Best Practices
1. **Use meaningful names** — `$productPrice` is far better than `$p` or `$x`. Descriptive names make code self-documenting.
2. **Initialize variables before use** — Always assign a value before reading a variable to avoid undefined variable notices.
3. **Prefer `isset()` over `empty()` for existence checks** — `empty()` has surprising behavior with falsy values like `0` and `"0"`. Use `isset()` when you just need to know if a variable exists.

## Summary
- PHP variables start with `$` and are case-sensitive.
- Variable names must begin with a letter or underscore and cannot contain hyphens or spaces.
- Use `isset()` to check if a variable exists, `empty()` to check for empty values, and `unset()` to destroy a variable.
- Always initialize variables and use descriptive names.

## Code Examples

**Practical example of declaring variables with different types and outputting them — notice how double-quoted strings interpolate variables**

```php
<?php
// Variable declaration and usage in a real scenario
$productName = "Laptop";
$productPrice = 999.99;
$inStock = true;
$quantity = 5;

// String interpolation works inside double quotes
echo "Product: $productName\n";
echo "Price: \$$productPrice\n";
echo "In Stock: " . ($inStock ? "Yes" : "No") . "\n";
echo "Available: $quantity units";

// Output:
// Product: Laptop
// Price: $999.99
// In Stock: Yes
// Available: 5 units
?>
```


## Resources

- [PHP Variables](https://www.php.net/manual/en/language.variables.php) — Complete guide to PHP variables

---

> 📘 *This lesson is part of the [PHP Essentials](https://stanza.dev/courses/php-essentials) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*