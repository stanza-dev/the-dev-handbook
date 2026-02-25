---
source_course: "php-essentials"
source_lesson: "php-essentials-type-declarations"
---

# Type Declarations in Modern PHP

## Introduction
Modern PHP (8.x) supports strict type declarations for function parameters, return values, and class properties. Type declarations make your code more reliable, self-documenting, and easier for IDEs to analyze. This lesson shows you how to use them effectively.

## Key Concepts
- **`declare(strict_types=1)`**: A per-file directive that forces strict type checking, preventing automatic type coercion.
- **Parameter Type**: Declares what type a function argument must be (e.g., `string $name`).
- **Return Type**: Declares what type a function must return, placed after the parameter list with a colon (e.g., `): int`).
- **Union Type (`int|string`)**: Allows a parameter or return value to accept multiple types (PHP 8.0+).
- **Nullable Type (`?int`)**: Indicates a value can be the specified type or `null`.

## Real World Context
In professional PHP codebases, type declarations are standard practice. They catch entire categories of bugs at call time rather than deep inside your functions. They also power IDE features like autocompletion and inline error detection. Laravel, Symfony, and all major frameworks use strict type declarations throughout their codebases.

## Deep Dive

### Enabling Strict Types

Add this directive at the very top of your file to enforce strict type checking:

```php
<?php
declare(strict_types=1);
```

Without strict types, PHP silently coerces values (e.g., passing `"42"` to an `int` parameter works). With strict types enabled, PHP throws a `TypeError` for any type mismatch.

### Function Parameter Types

Declare the expected type before each parameter name:

```php
<?php
declare(strict_types=1);

function greet(string $name): void {
    echo "Hello, $name!";
}

greet("Alice");  // Works
// greet(123);   // TypeError! Expected string, got int
```

The `void` return type indicates the function does not return a value.

### Return Type Declarations

Specify the return type after the parameter list with a colon:

```php
<?php
function add(int $a, int $b): int {
    return $a + $b;
}

function getUser(int $id): ?User {  // ? allows null
    return $id > 0 ? new User() : null;
}
```

The `?` prefix (nullable) means the function can return either the specified type or `null`.

### Union Types (PHP 8.0+)

Accept multiple types using the pipe (`|`) syntax:

```php
<?php
function processId(int|string $id): void {
    echo "Processing: $id";
}

processId(42);       // Works
processId("ABC123"); // Works
```

Union types are useful when a value can legitimately be more than one type.

### Nullable Types

Two equivalent ways to allow null:

```php
<?php
function find(?int $id): ?string {
    // $id can be int or null
    // Returns string or null
}

// PHP 8.0+ union syntax (equivalent)
function find(int|null $id): string|null {
    // Same as above
}
```

Both syntaxes are valid. The `?` shorthand is common for single types; the union syntax is used when there are already multiple types.

### Property Types (PHP 7.4+)

Class properties can also have type declarations:

```php
<?php
class Product {
    public string $name;
    public float $price;
    public ?string $description = null;
    private int $stock = 0;
}
```

Typed properties must be initialized before access, either in the declaration or in the constructor.

## Common Pitfalls
1. **Forgetting `declare(strict_types=1)`** — Without this directive, PHP still performs type coercion even with type declarations. Strict mode only applies to the file where it is declared.
2. **Placing `declare` after other code** — The `declare(strict_types=1)` statement must be the very first statement in the file (after the `<?php` tag). Placing it anywhere else causes a compile error.
3. **Mixing nullable syntax** — `?int` and `int|null` are equivalent, but mixing styles in the same codebase creates inconsistency. Pick one convention.

## Best Practices
1. **Always enable strict types** — Add `declare(strict_types=1)` to every PHP file. It catches bugs early and makes behavior predictable.
2. **Type everything** — Add types to all function parameters, return values, and class properties. This serves as both documentation and a safety net.
3. **Prefer specific types over `mixed`** — Use `mixed` only when a value truly can be any type. Specific types give better IDE support and error messages.

## Summary
- `declare(strict_types=1)` enforces strict type checking per file.
- Use parameter types, return types, and property types to catch bugs at call time.
- Union types (`int|string`) and nullable types (`?int`) handle multiple possible types.
- Strict types are standard practice in modern PHP codebases.

## Code Examples

**A Calculator class using union types and nullable returns — strict_types ensures type safety throughout**

```php
<?php
declare(strict_types=1);

// A class demonstrating modern PHP type declarations
class Calculator {
    public function add(int|float $a, int|float $b): float {
        return (float) ($a + $b);
    }

    public function divide(float $a, float $b): ?float {
        if ($b === 0.0) {
            return null;  // Avoid division by zero
        }
        return $a / $b;
    }
}

$calc = new Calculator();
echo $calc->add(5, 3.5);      // 8.5
echo $calc->divide(10, 0);    // null (no error)
echo $calc->divide(10, 3);    // 3.3333333333333
?>
```


## Resources

- [Type Declarations](https://www.php.net/manual/en/language.types.declarations.php) — Official guide to PHP type declarations

---

> 📘 *This lesson is part of the [PHP Essentials](https://stanza.dev/courses/php-essentials) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*