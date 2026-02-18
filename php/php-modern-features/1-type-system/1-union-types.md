---
source_course: "php-modern-features"
source_lesson: "php-modern-features-union-types"
---

# Union Types (PHP 8.0+)

Union types allow a value to be one of several types. This brings more flexibility while maintaining type safety.

## Basic Syntax

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

## Union Types with Properties

```php
<?php
class ApiResponse {
    public int|string $id;
    public array|object $data;
    public string|null $error;
}
```

## Return Type Unions

```php
<?php
function findUser(int $id): User|null {
    // Returns User or null
    return $this->users[$id] ?? null;
}

function getValue(string $key): string|int|float|bool {
    // Can return multiple scalar types
    return $this->config[$key];
}
```

## The `false` Pseudo-Type

PHP 8.0 allows `false` as a union member (for legacy API compatibility):

```php
<?php
function search(array $haystack, mixed $needle): int|false {
    $index = array_search($needle, $haystack);
    return $index;  // Returns index or false
}
```

## The `null` in Unions

```php
<?php
// These are equivalent:
function example1(?string $name): void {}
function example2(string|null $name): void {}

// But union syntax allows more types:
function example3(string|int|null $value): void {}
```

## Type Narrowing

```php
<?php
function process(int|string|array $data): string {
    // Type narrowing with conditionals
    if (is_array($data)) {
        return implode(', ', $data);
    }
    
    if (is_int($data)) {
        return (string) $data;
    }
    
    return $data;  // Must be string
}
```

## Union Types with Interfaces

```php
<?php
interface Stringable {
    public function __toString(): string;
}

function render(string|Stringable $content): string {
    return (string) $content;
}
```

## Code Examples

**Configuration class using union types**

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