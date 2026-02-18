---
source_course: "php-modern-features"
source_lesson: "php-modern-features-special-types"
---

# Special Types in PHP 8

PHP 8 introduced and refined special types for precise type declarations.

## The `mixed` Type (PHP 8.0+)

Accepts any value - equivalent to `array|bool|callable|int|float|null|object|resource|string`:

```php
<?php
function debug(mixed $value): void {
    var_dump($value);
}

debug(42);           // OK
debug('hello');      // OK
debug([1, 2, 3]);    // OK
debug(null);         // OK

// Unlike no type declaration, mixed is explicit
function process(mixed $data): mixed {
    // Explicitly saying "anything goes"
    return $data;
}
```

## The `void` Return Type

Indicates a function returns nothing:

```php
<?php
function logMessage(string $message): void {
    file_put_contents('log.txt', $message, FILE_APPEND);
    // No return statement, or just 'return;'
}

// Cannot return a value!
function invalid(): void {
    return 'oops';  // Error!
}
```

## The `never` Return Type (PHP 8.1+)

Indicates a function never returns (always throws or exits):

```php
<?php
function abort(string $message): never {
    throw new RuntimeException($message);
}

function redirect(string $url): never {
    header("Location: $url");
    exit;
}

function notImplemented(): never {
    throw new LogicException('Not implemented yet');
}

// Useful for static analysis
function processOrFail(mixed $data): string {
    if (!is_string($data)) {
        abort('Invalid data type');  // never returns
    }
    
    return $data;  // Analyzer knows this line is only reached if $data is string
}
```

## void vs never

| Type | Returns | Completes Normally |
|------|---------|--------------------|
| `void` | Nothing | Yes |
| `never` | Nothing | No (always throws/exits) |

## The `null` Type (PHP 8.2+)

Can be used as a standalone type:

```php
<?php
class NullLogger {
    public function log(string $message): null {
        // Do nothing, return null explicitly
        return null;
    }
}
```

## The `true` and `false` Types (PHP 8.2+)

```php
<?php
function alwaysTrue(): true {
    return true;
}

function alwaysFalse(): false {
    return false;
}

// Useful for methods that never fail
interface ConnectionPool {
    // Returns true on success, never returns false
    public function release(Connection $conn): true;
}
```

## Code Examples

**Router using void and never types**

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