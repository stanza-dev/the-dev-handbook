---
source_course: "php-modern-features"
source_lesson: "php-modern-features-type-narrowing"
---

# Type Narrowing & Assertions

PHP 8's type system works with control flow analysis to narrow types based on conditions.

## Automatic Type Narrowing

```php
<?php
function process(string|int|null $value): string
{
    // At this point: string|int|null
    
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

## Narrowing with instanceof

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
        // $item is now Renderable
        $output = $item->render();
    }
    
    if ($item instanceof Cacheable) {
        // $item is now Cacheable
        $key = $item->getCacheKey();
    }
    
    return $output;
}
```

## Assert for Static Analysis

```php
<?php
/**
 * @param mixed $value
 * @return User
 * @throws InvalidArgumentException
 */
function assertUser(mixed $value): User
{
    if (!$value instanceof User) {
        throw new InvalidArgumentException('Expected User');
    }
    
    return $value;  // Type is narrowed to User
}

// Usage
$data = $cache->get('user');
$user = assertUser($data);
// $user is definitely User now
```

## Narrowing with match

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

## The assert() Function

```php
<?php
// Development assertions
assert($user instanceof User, 'Expected User instance');

// In production, assertions can be disabled:
// zend.assertions = -1 in php.ini

// Custom assertion handler
set_assert_callback(function(string $file, int $line, ?string $assertion) {
    throw new AssertionError("Assertion failed: $assertion at $file:$line");
});
```

## PHPStan/Psalm Assertions

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

## Resources

- [Type Declarations](https://www.php.net/manual/en/language.types.declarations.php) — PHP type declarations documentation

---

> 📘 *This lesson is part of the [Modern PHP 8.x: Latest Language Features](https://stanza.dev/courses/php-modern-features) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*