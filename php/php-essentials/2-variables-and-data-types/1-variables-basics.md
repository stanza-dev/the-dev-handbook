---
source_course: "php-essentials"
source_lesson: "php-essentials-variables-basics"
---

# Understanding Variables

Variables in PHP are containers for storing data values. They're fundamental to every PHP program.

## Variable Syntax

In PHP, all variables start with a dollar sign (`$`):

```php
<?php
$name = "John";      // String
$age = 25;           // Integer
$price = 19.99;      // Float
$isActive = true;    // Boolean
```

## Variable Naming Rules

1. Must start with a letter or underscore (`_`)
2. Can contain letters, numbers, and underscores
3. Case-sensitive (`$name` ≠ `$Name`)
4. Cannot start with a number

```php
<?php
// Valid variable names
$userName = "Alice";
$user_name = "Bob";
$_private = "secret";
$item1 = "first";

// Invalid variable names
$1item = "wrong";     // Cannot start with number
$user-name = "wrong"; // Hyphens not allowed
```

## Variable Assignment

Use the assignment operator (`=`) to give variables values:

```php
<?php
$message = "Hello";        // Assign a value
$message = "Goodbye";      // Reassign (overwrite)

$a = $b = $c = 10;         // Multiple assignment
```

## Variable Variables

PHP allows variable variable names (use sparingly):

```php
<?php
$varName = "greeting";
$$varName = "Hello!";  // Creates $greeting

echo $greeting;  // Outputs: Hello!
```

## Checking Variables

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

## Code Examples

**Practical example of different variable types**

```php
<?php
// Variable declaration and usage
$productName = "Laptop";
$productPrice = 999.99;
$inStock = true;
$quantity = 5;

echo "Product: $productName\n";
echo "Price: $$productPrice\n";
echo "In Stock: " . ($inStock ? "Yes" : "No") . "\n";
echo "Available: $quantity units";
?>
```


## Resources

- [PHP Variables](https://www.php.net/manual/en/language.variables.php) — Complete guide to PHP variables

---

> 📘 *This lesson is part of the [PHP Essentials](https://stanza.dev/courses/php-essentials) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*