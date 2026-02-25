---
source_course: "php-modern-features"
source_lesson: "php-modern-features-special-types"
---

# Special Types: mixed, never, void

## Introduction
PHP 8 introduced and refined several special types that express precise intent about what a function accepts or returns. Understanding `mixed`, `void`, and `never` is essential for writing clear, self-documenting APIs.

## Key Concepts
- **`mixed`**: Accepts any value — the explicit "I accept anything" type (PHP 8.0+).
- **`void`**: The function returns nothing and completes normally (PHP 7.1+).
- **`never`**: The function never returns — it always throws or exits (PHP 8.1+).
- **`null` as standalone type**: Can be used as a return type on its own (PHP 8.2+).
- **`true`/`false` literal types**: Standalone boolean literal types (PHP 8.2+).

## Real World Context
Choosing the right special type communicates intent to both humans and static analyzers. A function typed `never` tells your IDE that code after calling it is unreachable. A function typed `void` tells callers not to use its return value. These distinctions prevent bugs in real codebases.

## Deep Dive

### The `mixed` Type (PHP 8.0+)

The `mixed` type accepts any value. It is equivalent to `array|bool|callable|int|float|null|object|resource|string`:

```php
<?php
function debug(mixed $value): void {
    var_dump($value);
}

debug(42);           // OK
debug('hello');      // OK
debug([1, 2, 3]);    // OK
debug(null);         // OK
```

Unlike omitting the type entirely, `mixed` is an explicit declaration. It tells readers and tools that accepting any type was a deliberate choice, not an oversight.

### The `void` Return Type

A `void` function returns nothing and completes normally:

```php
<?php
function logMessage(string $message): void {
    file_put_contents('log.txt', $message, FILE_APPEND);
    // No return statement, or just 'return;'
}

function invalid(): void {
    return 'oops';  // Error! Cannot return a value from void
}
```

The function runs to completion but its return value is always `null` and should not be used.

### The `never` Return Type (PHP 8.1+)

A `never` function never completes normally — it always throws an exception or calls `exit`:

```php
<?php
function abort(string $message): never {
    throw new RuntimeException($message);
}

function redirect(string $url): never {
    header("Location: $url");
    exit;
}
```

This is powerful for static analysis. Code after a `never` call is provably unreachable:

```php
<?php
function processOrFail(mixed $data): string {
    if (!is_string($data)) {
        abort('Invalid data type');  // never returns
    }
    return $data;  // Analyzer knows $data is string here
}
```

The analyzer understands that if `abort()` is called, execution stops, so `$data` must be a string on the last line.

### void vs never

| Type | Returns | Completes Normally |
|------|---------|--------------------|
| `void` | Nothing | Yes — function finishes |
| `never` | Nothing | No — always throws or exits |

### Standalone `null` Type (PHP 8.2+)

The `null` type can be used on its own as a return type:

```php
<?php
class NullLogger {
    public function log(string $message): null {
        return null;  // Explicitly returns null, nothing else
    }
}
```

### Literal `true` and `false` Types (PHP 8.2+)

These standalone boolean literal types are useful for methods that always succeed or always fail:

```php
<?php
interface ConnectionPool {
    public function release(Connection $conn): true;
}

function alwaysFalse(): false {
    return false;
}
```

This is more precise than returning `bool` when the outcome is guaranteed.

## Common Pitfalls
1. **Confusing `void` and `never`** — Use `void` when the function completes but returns nothing. Use `never` when the function never completes (always throws or exits).
2. **Using `mixed` as a lazy escape** — `mixed` should be a deliberate choice when any type is genuinely acceptable, not a shortcut to avoid thinking about types.

## Best Practices
1. **Use `never` for error helpers and redirects** — Functions like `abort()`, `redirect()`, and `notFound()` should return `never` so static analyzers can prune unreachable code paths.
2. **Prefer specific types over `mixed`** — If a function only actually handles strings and integers, use `string|int` instead of `mixed`.

## Summary
- `mixed` explicitly accepts any type — use it deliberately, not as a shortcut.
- `void` means the function returns nothing and completes normally.
- `never` means the function never returns — it always throws or exits.
- `null`, `true`, and `false` are standalone types in PHP 8.2+.
- Choosing the right special type helps static analyzers catch unreachable code and type errors.

## Code Examples

**Router using void and never types to express intent clearly**

```php
<?php
declare(strict_types=1);

// Practical use of special types
class Router {
    private array $routes = [];
    
    public function add(string $path, callable $handler): void {
        $this->routes[$path] = $handler;
    }
    
    public function dispatch(string $path): mixed {
        $handler = $this->routes[$path] ?? null;
        
        if ($handler === null) {
            $this->notFound($path);
        }
        
        return $handler();
    }
    
    private function notFound(string $path): never {
        http_response_code(404);
        echo "Page not found: $path";
        exit;
    }
}

$router = new Router();
$router->add('/', fn() => 'Home');
$router->add('/about', fn() => 'About');
?>
```


## Resources

- [Return Type Declarations](https://www.php.net/manual/en/functions.returning-values.php#functions.returning-values.type-declaration) — Documentation for return types including void and never

---

> 📘 *This lesson is part of the [Modern PHP 8.x: Latest Language Features](https://stanza.dev/courses/php-modern-features) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*