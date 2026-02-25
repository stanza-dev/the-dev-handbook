---
source_course: "php-essentials"
source_lesson: "php-essentials-data-types"
---

# PHP Data Types

## Introduction
PHP supports ten primitive data types grouped into three categories: scalar, compound, and special types. Understanding these types is essential for writing correct programs, debugging type-related issues, and taking advantage of PHP's type system.

## Key Concepts
- **Scalar Types**: Hold a single value — `string`, `int`, `float`, and `bool`.
- **Compound Types**: Hold multiple values or complex structures — `array`, `object`, `callable`, and `iterable`.
- **Special Types**: `null` (no value) and `resource` (external reference like a file handle).
- **Type Juggling**: PHP automatically converts types in certain contexts (e.g., `"5" + 3` produces `8`).

## Real World Context
Every variable in PHP has a type, even if you never declare one explicitly. When you read user input from a form, it arrives as a string. When you query a database, numeric columns may come back as strings too. Knowing your types — and how PHP converts between them — prevents bugs like treating `"0"` as truthy or comparing a string to an integer with unexpected results.

## Deep Dive

### Scalar Types (Single Values)

**String** — a sequence of characters:

```php
<?php
$single = 'Hello';           // Single quotes (literal)
$double = "Hello, $name";    // Double quotes (interpolation)
$heredoc = <<<EOT
Multi-line
string here
EOT;
```

Single-quoted strings treat everything literally. Double-quoted strings parse variables and escape sequences like `\n`.

**Integer** — whole numbers (positive or negative):

```php
<?php
$decimal = 42;            // Decimal
$negative = -17;          // Negative
$octal = 0755;            // Octal (starts with 0)
$hex = 0xFF;              // Hexadecimal (starts with 0x)
$binary = 0b1010;         // Binary (starts with 0b)
$underscore = 1_000_000;  // PHP 7.4+ readability
```

The underscore separator (`1_000_000`) makes large numbers easier to read without affecting the value.

**Float (Double)** — decimal numbers:

```php
<?php
$price = 19.99;
$scientific = 1.2e3;  // 1200
$negative = -0.5;
```

**Boolean** — true or false:

```php
<?php
$isValid = true;
$hasError = false;

// Falsy values in PHP:
// false, 0, 0.0, "", "0", [], null
```

Understanding which values are falsy is critical for writing correct conditionals.

### Compound Types

**Array** — ordered maps holding multiple values:

```php
<?php
$indexed = [1, 2, 3];
$associative = ["name" => "John", "age" => 30];
```

**Object** — instances of classes:

```php
<?php
class User {
    public string $name;
}
$user = new User();
```

**Callable** — anything that can be invoked as a function:

```php
<?php
$callback = function($x) { return $x * 2; };
```

### Special Types

**NULL** represents the absence of a value:

```php
<?php
$nothing = null;
```

**Resource** references external resources like file handles and database connections.

### Type Checking

PHP provides functions to inspect types at runtime:

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

The `gettype()` function returns a human-readable string, while `is_*` functions return a boolean for specific type checks.

## Common Pitfalls
1. **Trusting `"0"` as truthy** — In PHP, the string `"0"` is falsy, unlike most other languages. `if ("0")` evaluates to false.
2. **Loose comparison surprises** — `0 == "foo"` is `true` in PHP because the string gets coerced to `0`. Always use strict comparison (`===`) to avoid this.
3. **Floating-point precision** — `0.1 + 0.2 !== 0.3` due to IEEE 754 representation. Use `round()` or the `bcmath` extension for precise decimal math.

## Best Practices
1. **Use strict comparison (`===`)** — It checks both value and type, eliminating type juggling surprises.
2. **Use `var_dump()` for debugging, not `echo`** — `echo` converts everything to a string, hiding the actual type. `var_dump()` shows both type and value.
3. **Type-hint function parameters** — Declare expected types on function parameters and return values. This catches type errors early and serves as documentation.

## Summary
- PHP has 10 primitive types: 4 scalar, 4 compound, and 2 special.
- Scalar types (`string`, `int`, `float`, `bool`) hold single values.
- PHP performs automatic type juggling, which can cause unexpected behavior.
- Use strict comparison (`===`) and type-checking functions to write reliable code.

## Code Examples

**Demonstrating PHP's type checking functions — var_dump shows both type and value, making it invaluable for debugging**

```php
<?php
// Type checking demonstration
$values = [
    "Hello",      // string
    42,           // integer
    3.14,         // float
    true,         // boolean
    [1, 2, 3],   // array
    null          // null
];

foreach ($values as $value) {
    echo gettype($value) . ": ";
    var_dump($value);
}

// Output shows both the type name and the detailed value
// string: string(5) "Hello"
// integer: int(42)
// double: float(3.14)
// boolean: bool(true)
// array: array(3) { ... }
// NULL: NULL
?>
```


## Resources

- [PHP Types](https://www.php.net/manual/en/language.types.php) — Complete reference for all PHP data types

---

> 📘 *This lesson is part of the [PHP Essentials](https://stanza.dev/courses/php-essentials) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*