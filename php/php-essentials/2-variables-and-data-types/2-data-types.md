---
source_course: "php-essentials"
source_lesson: "php-essentials-data-types"
---

# PHP Data Types

PHP supports ten primitive data types, grouped into three categories.

## Scalar Types (Single Values)

### 1. String
A sequence of characters:
```php
<?php
$single = 'Hello';           // Single quotes (literal)
$double = "Hello, $name";    // Double quotes (interpolation)
$heredoc = <<<EOT
Multi-line
string here
EOT;
```

### 2. Integer
Whole numbers (positive or negative):
```php
<?php
$decimal = 42;        // Decimal
$negative = -17;      // Negative
$octal = 0755;        // Octal (starts with 0)
$hex = 0xFF;          // Hexadecimal (starts with 0x)
$binary = 0b1010;     // Binary (starts with 0b)
$underscore = 1_000_000;  // PHP 7.4+ readability
```

### 3. Float (Double)
Decimal numbers:
```php
<?php
$price = 19.99;
$scientific = 1.2e3;  // 1200
$negative = -0.5;
```

### 4. Boolean
True or false:
```php
<?php
$isValid = true;
$hasError = false;

// Falsy values in PHP:
// false, 0, 0.0, "", "0", [], null
```

## Compound Types

### 5. Array
```php
<?php
$indexed = [1, 2, 3];
$associative = ["name" => "John", "age" => 30];
```

### 6. Object
```php
<?php
class User {
    public string $name;
}
$user = new User();
```

### 7. Callable
Functions that can be called:
```php
<?php
$callback = function($x) { return $x * 2; };
```

### 8. Iterable
Anything that can be looped over (arrays, Traversable objects).

## Special Types

### 9. NULL
Represents no value:
```php
<?php
$nothing = null;
$uninitialized;  // Also null if accessed
```

### 10. Resource
References to external resources (file handles, database connections).

## Type Checking

```php
<?php
gettype($var);      // Returns type as string
is_string($var);    // Boolean check
is_int($var);
is_float($var);
is_bool($var);
is_array($var);
is_null($var);
```

## Code Examples

**Demonstrating PHP's type checking functions**

```php
<?php
// Type checking demonstration
$values = [
    "Hello",
    42,
    3.14,
    true,
    [1, 2, 3],
    null
];

foreach ($values as $value) {
    echo gettype($value) . ": ";
    var_dump($value);
    echo "\n";
}
?>
```


## Resources

- [PHP Types](https://www.php.net/manual/en/language.types.php) — Complete reference for all PHP data types

---

> 📘 *This lesson is part of the [PHP Essentials](https://stanza.dev/courses/php-essentials) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*