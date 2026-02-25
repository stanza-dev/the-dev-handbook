---
source_course: "php-modern-features"
source_lesson: "php-modern-features-union-types"
---

# Union Types

## Introduction
Union types, introduced in PHP 8.0, allow a value to accept one of several types. They bring flexibility while maintaining type safety, eliminating the need for docblock-only annotations that could not be enforced at runtime.

## Key Concepts
- **Union Type**: A type declaration using `|` that accepts any of the listed types, e.g. `int|string`.
- **Nullable Union**: Writing `string|null` is equivalent to `?string`, but the union syntax supports more than two types.
- **`false` Pseudo-Type**: PHP 8.0 allows `false` in unions for legacy APIs that return `false` on failure.

## Real World Context
Before union types, PHP developers relied on docblock annotations like `@param int|string $id` that were invisible to the runtime. Now the engine enforces these constraints, catching bugs earlier and making APIs self-documenting.

## Deep Dive

### Basic Syntax

Union types use the pipe `|` character between types:

```php
<?php
function processId(int|string $id): void {
    if (is_int($id)) {
        echo "Integer ID: $id";
    } else {
        echo "String ID: $id";
    }
}

processId(42);       // Integer ID: 42
processId("ABC123"); // String ID: ABC123
```

The function above accepts both integer and string arguments. PHP enforces this at runtime — passing a `bool` would throw a `TypeError`.

### Union Types with Properties

You can use union types on class properties as well:

```php
<?php
class ApiResponse {
    public int|string $id;
    public array|object $data;
    public string|null $error;
}
```

Each property is constrained to the declared types, enforced on assignment.

### Return Type Unions

Return types benefit from unions too:

```php
<?php
function findUser(int $id): User|null {
    return $this->users[$id] ?? null;
}

function getValue(string $key): string|int|float|bool {
    return $this->config[$key];
}
```

This tells callers exactly what types to expect, improving static analysis and IDE support.

### The `false` Pseudo-Type

Many legacy PHP functions return `false` on failure. The `false` pseudo-type formalizes this pattern:

```php
<?php
function search(array $haystack, mixed $needle): int|false {
    $index = array_search($needle, $haystack);
    return $index;  // Returns index or false
}
```

This makes the return type explicit rather than relying on docblock annotations.

### Nullable Shorthand vs Union

```php
<?php
// These are equivalent for single nullable types:
function example1(?string $name): void {}
function example2(string|null $name): void {}

// But union syntax is required for multi-type nullable:
function example3(string|int|null $value): void {}
```

The `?` shorthand only works with a single type. For multiple types plus null, use the full union syntax.

### Type Narrowing with Unions

```php
<?php
function process(int|string|array $data): string {
    if (is_array($data)) {
        return implode(', ', $data);
    }
    if (is_int($data)) {
        return (string) $data;
    }
    return $data;  // Must be string here
}
```

PHP and static analyzers like PHPStan narrow the type inside each branch, ensuring type-safe operations.

## Common Pitfalls
1. **Duplicate types in unions** — Writing `int|int` or `?string|null` (double null) causes a compile error. Each type must appear exactly once.
2. **Union with `void`** — `void` cannot appear in a union because a void function must return nothing. Use `null` in a union instead if you want a nullable return.

## Best Practices
1. **Prefer narrow unions** — `int|string` is better than `mixed`. The narrower the union, the more the type checker can help you.
2. **Use union types instead of docblocks** — Native union types are enforced at runtime and understood by all static analysis tools.

## Summary
- Union types use `|` to accept multiple types in parameters, returns, and properties.
- They replace docblock-only type annotations with runtime-enforced declarations.
- The `false` pseudo-type handles legacy PHP patterns cleanly.
- Nullable unions (`string|null`) replace the `?` shorthand when more than one type is involved.
- Type narrowing with `is_*()` functions works naturally inside union-typed code.

## Code Examples

**Configuration class using union types for flexible value storage while maintaining type safety**

```php
<?php
declare(strict_types=1);

// Real-world: Configuration handler with union types
class Config {
    private array $settings = [];
    
    public function set(string $key, string|int|float|bool|array $value): void {
        $this->settings[$key] = $value;
    }
    
    public function get(string $key): string|int|float|bool|array|null {
        return $this->settings[$key] ?? null;
    }
    
    public function getString(string $key): string|null {
        $value = $this->get($key);
        return is_string($value) ? $value : null;
    }
}

$config = new Config();
$config->set('app.name', 'MyApp');
$config->set('app.debug', true);
$config->set('app.max_users', 100);

echo $config->getString('app.name');  // MyApp
?>
```


## Resources

- [Union Types RFC](https://www.php.net/manual/en/language.types.declarations.php#language.types.declarations.union) — Official documentation for PHP 8 union types

---

> 📘 *This lesson is part of the [Modern PHP 8.x: Latest Language Features](https://stanza.dev/courses/php-modern-features) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*