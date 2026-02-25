---
source_course: "php-essentials"
source_lesson: "php-essentials-constants"
---

# Constants in PHP

## Introduction
Constants are values that cannot change once defined. They are ideal for configuration settings, fixed data, and magic numbers that should be named for clarity. This lesson covers the two ways to define constants, class constants, typed constants in PHP 8.3+, and the special magic constants.

## Key Concepts
- **Constant**: A named value that cannot be changed after it is defined. Unlike variables, constants do not use the `$` prefix.
- **`const` keyword**: Defines a constant at compile time. Preferred in modern PHP.
- **`define()` function**: Defines a constant at runtime. Required when the definition is conditional.
- **Magic Constants**: Special constants like `__FILE__` and `__DIR__` whose values change depending on where they are used in the code.

## Real World Context
Every application has values that should never change: database credentials, API rate limits, feature flags, HTTP status codes. Using constants instead of hard-coded values prevents accidental modification and makes your code self-documenting. Class constants are especially common in frameworks for defining enumerations and configuration keys.

## Deep Dive

### Using `const` (Preferred in PHP 8+)

The `const` keyword defines constants at compile time:

```php
<?php
const APP_NAME = "My Application";
const VERSION = "2.0";
const MAX_USERS = 100;
const FEATURES = ["auth", "api", "admin"];  // Arrays allowed
```

Constants defined with `const` are resolved at compile time, making them slightly faster than `define()`.

### Using `define()` Function

The `define()` function creates constants at runtime, which allows conditional definitions:

```php
<?php
define("APP_NAME", "My Application");
define("DEBUG_MODE", true);

// Conditional definition (only possible with define)
if (!defined("MAX_RETRIES")) {
    define("MAX_RETRIES", 3);
}
```

Use `defined()` to check whether a constant already exists before defining it.

### `const` vs `define()`

Here is a comparison to help you choose:

| Feature | `const` | `define()` |
|---------|---------|------------|
| Scope | Compile-time | Runtime |
| Conditional | No | Yes |
| In classes | Yes | No |
| Speed | Faster | Slower |

In most cases, prefer `const`. Use `define()` only when you need conditional or dynamic definitions.

### Class Constants

Constants can be defined inside classes, providing scoped, organized values:

```php
<?php
class HttpStatus {
    public const OK = 200;
    public const NOT_FOUND = 404;
    public const SERVER_ERROR = 500;

    private const SECRET = "hidden";  // Visibility control
}

echo HttpStatus::OK;  // 200
```

Class constants are accessed with the `::` (scope resolution) operator and can have visibility modifiers.

### Typed Constants (PHP 8.3+)

PHP 8.3 introduced type declarations for class constants:

```php
<?php
class Config {
    public const string APP_NAME = "MyApp";
    public const int MAX_SIZE = 1024;
    public const array ALLOWED = ["a", "b"];
}
```

Typed constants prevent accidentally defining a constant with the wrong type.

### Magic Constants

PHP provides special constants whose values change based on context:

```php
<?php
echo __FILE__;      // Full path to current file
echo __DIR__;       // Directory of current file
echo __LINE__;      // Current line number
echo __FUNCTION__;  // Function name
echo __CLASS__;     // Class name
echo __METHOD__;    // Class method name
echo __NAMESPACE__; // Current namespace
```

Magic constants are resolved at compile time and are invaluable for debugging and logging.

## Common Pitfalls
1. **Trying to redefine a constant** — Once defined, a constant cannot be changed. Attempting to redefine it silently does nothing (with `define()`) or causes an error (with `const`).
2. **Using `$` with constants** — Constants do not use the dollar sign. Writing `$APP_NAME` creates a variable, not a reference to the constant `APP_NAME`.
3. **Forgetting scope with `const`** — `const` inside a function creates a constant in the function's namespace scope, which may not be what you expect. Global constants should be defined at the top level.

## Best Practices
1. **Use UPPER_CASE for constant names** — This is the universal convention that visually distinguishes constants from variables.
2. **Group related constants in classes** — Instead of many global constants, organize them into classes: `HttpStatus::OK`, `Config::MAX_RETRIES`.
3. **Prefer `const` for simple values, `define()` for conditional** — Only use `define()` when you need runtime logic to decide the value.

## Summary
- Constants are immutable values defined with `const` or `define()`.
- `const` is compile-time and preferred; `define()` is runtime and supports conditional definitions.
- Class constants provide scoped, organized values with visibility control.
- Magic constants like `__FILE__` and `__DIR__` change based on context.
- Use UPPER_CASE naming and group related constants in classes.

## Code Examples

**Organizing constants as globals and class members — class constants are accessed via the :: operator**

```php
<?php
// Configuration constants
const DB_HOST = "localhost";
const DB_NAME = "myapp";

// Application constants organized in a class
class App {
    public const VERSION = "1.0.0";
    public const ENV = "development";

    public const ENVIRONMENTS = [
        "development",
        "staging",
        "production"
    ];
}

echo "App Version: " . App::VERSION;  // App Version: 1.0.0
echo "Running on: " . DB_HOST;        // Running on: localhost

// Magic constant for debugging
echo "This file: " . __FILE__;
?>
```


## Resources

- [PHP Constants](https://www.php.net/manual/en/language.constants.php) — Official documentation for PHP constants

---

> 📘 *This lesson is part of the [PHP Essentials](https://stanza.dev/courses/php-essentials) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*