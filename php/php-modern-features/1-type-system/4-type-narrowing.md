---
source_course: "php-modern-features"
source_lesson: "php-modern-features-type-narrowing"
---

# Type Narrowing & Assertions

## Introduction
When working with union types, you need to narrow a broad type down to a specific one before performing type-specific operations. PHP 8's type system works with control flow analysis to narrow types based on conditions, making your code both safe and expressive.

## Key Concepts
- **Type Narrowing**: The process of reducing a union type to a more specific type using conditionals.
- **`instanceof` Check**: Tests whether an object implements a specific class or interface.
- **Assertion Functions**: Helper functions that throw when a type check fails, allowing static analyzers to narrow types after the call.

## Real World Context
Every time you receive `mixed` data from a cache, a request payload, or an external API, you must narrow it to a concrete type before using it. Type narrowing is the bridge between loosely-typed input boundaries and strongly-typed business logic.

## Deep Dive

### Automatic Type Narrowing

PHP narrows types automatically based on control flow:

```php
<?php
function process(string|int|null $value): string
{
    if ($value === null) {
        return 'empty';
    }
    // Now: string|int (null excluded)
    
    if (is_string($value)) {
        return strtoupper($value);  // string methods OK
    }
    // Now: int only
    
    return (string) $value;  // int to string
}
```

After each condition, the type is narrowed. Once `null` is excluded, the remaining type is `string|int`. After the `is_string()` check passes, only `int` remains.

### Narrowing with instanceof

The `instanceof` operator narrows object types:

```php
<?php
interface Renderable {
    public function render(): string;
}

interface Cacheable {
    public function getCacheKey(): string;
}

function display(object $item): string
{
    $output = '';
    
    if ($item instanceof Renderable) {
        $output = $item->render();  // $item is Renderable here
    }
    
    if ($item instanceof Cacheable) {
        $key = $item->getCacheKey();  // $item is Cacheable here
    }
    
    return $output;
}
```

Inside each `instanceof` block, you can safely call the interface methods.

### Assert Functions for Static Analysis

Assertion functions throw on invalid types, allowing analyzers to narrow the type after the call:

```php
<?php
function assertUser(mixed $value): User
{
    if (!$value instanceof User) {
        throw new InvalidArgumentException('Expected User');
    }
    return $value;  // Type is narrowed to User
}

$data = $cache->get('user');
$user = assertUser($data);
// $user is definitely User now
```

After `assertUser()` returns, both the runtime and static analyzers know the type is `User`.

### Narrowing with match

The `match` expression works with type-checking functions:

```php
<?php
function handle(string|int|array $input): string
{
    return match(true) {
        is_string($input) => "String: $input",
        is_int($input) => "Integer: $input",
        is_array($input) => "Array: " . count($input) . " items",
    };
}
```

The `match(true)` pattern evaluates each arm's condition and returns the result of the first match.

### PHPStan/Psalm Assertions

Static analyzers support special annotations for assertion functions:

```php
<?php
/**
 * @phpstan-assert User $value
 * @psalm-assert User $value
 */
function assertIsUser(mixed $value): void
{
    if (!$value instanceof User) {
        throw new TypeError('Expected User');
    }
}

function processUser(mixed $data): void
{
    assertIsUser($data);
    // Static analyzers now know $data is User
    echo $data->getName();  // No error!
}
```

These annotations tell PHPStan and Psalm to narrow the type after the assertion call succeeds.

## Common Pitfalls
1. **Forgetting the early return after null checks** — Without an early return or throw, the null type is not excluded from the remaining code. Always return, throw, or continue after a null check.
2. **Using `gettype()` for narrowing** — Static analyzers do not understand `gettype()` for type narrowing. Use `is_string()`, `is_int()`, `instanceof`, etc. instead.

## Best Practices
1. **Narrow at system boundaries** — Validate and narrow types as early as possible, at the entry points of your application (controllers, command handlers), so the inner layers work with concrete types.
2. **Use assertion functions for reusable checks** — Extract common type assertions into helper functions annotated with `@phpstan-assert` so the narrowing is reusable across the codebase.

## Summary
- Type narrowing reduces union types to specific types using conditionals.
- `is_*()` functions and `instanceof` enable automatic narrowing.
- Assertion functions throw on invalid types, narrowing for subsequent code.
- PHPStan and Psalm support `@phpstan-assert` annotations for custom assertion functions.
- Always narrow types at system boundaries for maximum type safety.

## Code Examples

**Request validator using type narrowing to safely extract typed values from untyped input arrays**

```php
<?php
declare(strict_types=1);

// Type-safe request data extraction with narrowing
class RequestValidator {
    public function __construct(private array $data) {}
    
    public function requireString(string $key): string {
        $value = $this->data[$key] ?? null;
        if (!is_string($value)) {
            throw new InvalidArgumentException("$key must be a string");
        }
        return $value;  // Narrowed to string
    }
    
    public function requireInt(string $key): int {
        $value = $this->data[$key] ?? null;
        if (!is_int($value)) {
            throw new InvalidArgumentException("$key must be an integer");
        }
        return $value;  // Narrowed to int
    }
    
    public function optionalString(string $key): ?string {
        $value = $this->data[$key] ?? null;
        if ($value === null) {
            return null;  // null path
        }
        if (!is_string($value)) {
            throw new InvalidArgumentException("$key must be a string or null");
        }
        return $value;  // Narrowed to string
    }
}

$validator = new RequestValidator($_POST);
$name = $validator->requireString('name');   // string
$age = $validator->requireInt('age');        // int
$bio = $validator->optionalString('bio');    // ?string
?>
```


## Resources

- [Type Declarations](https://www.php.net/manual/en/language.types.declarations.php) — PHP type declarations documentation

---

> 📘 *This lesson is part of the [Modern PHP 8.x: Latest Language Features](https://stanza.dev/courses/php-modern-features) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*