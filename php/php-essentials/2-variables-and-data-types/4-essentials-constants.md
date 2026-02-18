---
source_course: "php-essentials"
source_lesson: "php-essentials-constants"
---

# Constants in PHP

Constants are like variables, but their values cannot change once defined. They're ideal for configuration values and fixed data.

## Defining Constants

### Using `const` (Preferred in PHP 8+)

```php
<?php
const APP_NAME = "My Application";
const VERSION = "2.0";
const MAX_USERS = 100;
const FEATURES = ["auth", "api", "admin"];  // Arrays allowed
```

### Using `define()` Function

```php
<?php
define("APP_NAME", "My Application");
define("DEBUG_MODE", true);

// Conditional definition (only with define)
if (!defined("MAX_RETRIES")) {
    define("MAX_RETRIES", 3);
}
```

## `const` vs `define()`

| Feature | `const` | `define()` |
|---------|---------|------------|
| Scope | Compile-time | Runtime |
| Conditional | No | Yes |
| In classes | Yes | No |
| Case-insensitive | No | No (deprecated) |
| Speed | Faster | Slower |

## Class Constants

```php
<?php
class HttpStatus {
    public const OK = 200;
    public const NOT_FOUND = 404;
    public const SERVER_ERROR = 500;
    
    private const SECRET = "hidden";  // Visibility
}

echo HttpStatus::OK;  // 200
```

## Typed Constants (PHP 8.3+)

```php
<?php
class Config {
    public const string APP_NAME = "MyApp";
    public const int MAX_SIZE = 1024;
    public const array ALLOWED = ["a", "b"];
}
```

## Magic Constants

PHP provides special constants that change based on context:

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

## Best Practices

1. Use `UPPER_CASE` for constant names
2. Group related constants in classes
3. Use `const` for simple values, `define()` for conditional
4. Prefer class constants over global constants

## Code Examples

**Organizing constants in PHP applications**

```php
<?php
// Configuration constants
const DB_HOST = "localhost";
const DB_NAME = "myapp";

// Application constants in a class
class App {
    public const VERSION = "1.0.0";
    public const ENV = "development";
    
    public const ENVIRONMENTS = [
        "development",
        "staging", 
        "production"
    ];
}

echo "App Version: " . App::VERSION;
echo "Running on: " . DB_HOST;
?>
```


## Resources

- [PHP Constants](https://www.php.net/manual/en/language.constants.php) — Official documentation for PHP constants

---

> 📘 *This lesson is part of the [PHP Essentials](https://stanza.dev/courses/php-essentials) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*