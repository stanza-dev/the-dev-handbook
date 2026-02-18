---
source_course: "php-essentials"
source_lesson: "php-essentials-type-declarations"
---

# Type Declarations in Modern PHP

PHP 8+ supports strict type declarations, making your code more reliable and self-documenting.

## Enabling Strict Types

Add this at the top of your file:

```php
<?php
declare(strict_types=1);
```

With strict types, PHP will throw a `TypeError` if the wrong type is passed.

## Function Parameter Types

```php
<?php
declare(strict_types=1);

function greet(string $name): void {
    echo "Hello, $name!";
}

greet("Alice");  // Works
greet(123);      // TypeError!
```

## Return Type Declarations

```php
<?php
function add(int $a, int $b): int {
    return $a + $b;
}

function getUser(int $id): ?User {  // ? allows null
    return $id > 0 ? new User() : null;
}

function logMessage(string $msg): void {  // No return
    echo $msg;
}
```

## Union Types (PHP 8.0+)

Accept multiple types:

```php
<?php
function processId(int|string $id): void {
    echo "Processing: $id";
}

processId(42);       // Works
processId("ABC123"); // Works
```

## Nullable Types

Two ways to allow null:

```php
<?php
function find(?int $id): ?string {
    // $id can be int or null
    // Returns string or null
}

// PHP 8.0+ union syntax
function find(int|null $id): string|null {
    // Same as above
}
```

## Mixed Type (PHP 8.0+)

Accepts any type:

```php
<?php
function debug(mixed $value): void {
    var_dump($value);
}
```

## Property Types (PHP 7.4+)

```php
<?php
class Product {
    public string $name;
    public float $price;
    public ?string $description = null;
    private int $stock = 0;
}
```

## Benefits of Type Declarations

1. **Catch bugs early**: Errors at call time, not deep in code
2. **Self-documenting**: Code explains what it expects
3. **IDE support**: Better autocomplete and error detection
4. **Performance**: PHP can optimize typed code

## Code Examples

**Modern PHP class with type declarations**

```php
<?php
declare(strict_types=1);

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
echo $calc->divide(10, 0);    // null
?>
```


## Resources

- [Type Declarations](https://www.php.net/manual/en/language.types.declarations.php) — Official guide to PHP type declarations

---

> 📘 *This lesson is part of the [PHP Essentials](https://stanza.dev/courses/php-essentials) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*