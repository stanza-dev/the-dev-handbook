---
source_course: "php-modern-features"
source_lesson: "php-modern-features-named-arguments"
---

# Named Arguments

## Introduction
Named arguments, introduced in PHP 8.0, let you pass values to a function by parameter name instead of position. They make code self-documenting, allow skipping optional parameters, and pair perfectly with constructor promotion.

## Key Concepts
- **Named Argument**: A function argument passed as `paramName: value` instead of by position.
- **Skipping Optionals**: Named arguments let you specify only the parameters you need, skipping ones with defaults.
- **Argument Unpacking**: Spreading an associative array as named arguments with `...$array`.

## Real World Context
PHP has many built-in functions with five or more parameters where you only need to change the last one. Before named arguments, you had to pass all preceding defaults explicitly. Named arguments eliminate this boilerplate, especially with functions like `array_filter()`, `setcookie()`, and `json_encode()`.

## Deep Dive

### Basic Syntax

Pass arguments by name:

```php
<?php
function createUser(string $name, string $email, int $age, bool $active = true) {
    // ...
}

// Positional (old way)
createUser('John', 'john@example.com', 25, true);

// Named arguments - clearer intent
createUser(
    name: 'John',
    email: 'john@example.com',
    age: 25,
    active: true
);
```

Named arguments make the purpose of each value immediately clear without checking the function signature.

### Skipping Optional Parameters

This is where named arguments shine:

```php
<?php
function sendEmail(
    string $to,
    string $subject,
    string $body,
    string $from = 'noreply@example.com',
    array $cc = [],
    array $bcc = [],
    bool $html = false
) {}

// Only specify what you need
sendEmail(
    to: 'user@example.com',
    subject: 'Hello',
    body: 'Message content',
    html: true  // Skip $from, $cc, $bcc
);
```

Without named arguments, you would need to pass `'noreply@example.com', [], []` before `true`.

### Mixing Positional and Named

Positional arguments must come first:

```php
<?php
// OK: positional first, then named
createUser('John', 'john@example.com', age: 25, active: true);

// Error: cannot use positional after named
createUser(name: 'John', 'john@example.com');
```

This rule prevents ambiguity about which parameter receives which value.

### With Array Functions

Named arguments improve readability of built-in functions:

```php
<?php
$numbers = [3, 1, 4, 1, 5, 9];

// What does ARRAY_FILTER_USE_KEY mean here?
array_filter($numbers, fn($n) => $n > 2, ARRAY_FILTER_USE_KEY);

// Self-documenting with named args
array_filter(
    array: $numbers,
    callback: fn($n) => $n > 2,
    mode: ARRAY_FILTER_USE_BOTH
);
```

The named version makes the third argument's purpose immediately clear.

### Argument Unpacking with Names

Spread an associative array as named arguments:

```php
<?php
$args = [
    'name' => 'Alice',
    'email' => 'alice@example.com',
    'age' => 30,
];

createUser(...$args);
// Equivalent to:
createUser(name: 'Alice', email: 'alice@example.com', age: 30);
```

This is powerful for building function calls dynamically from configuration or request data.

## Common Pitfalls
1. **Renaming parameters breaks callers** — Named arguments create a dependency on parameter names. Renaming a parameter in a library function is now a breaking change.
2. **Using named arguments with variadic functions** — Named arguments and variadic parameters (`...$args`) have complex interaction rules. Stick to positional arguments for variadic functions.

## Best Practices
1. **Use named arguments for boolean and configuration parameters** — `createUser(active: true)` is much clearer than `createUser('John', 'john@ex.com', 25, true)` where the `true` meaning is unclear.
2. **Treat parameter names as part of your API** — Since callers depend on names, choose descriptive names and avoid renaming them in public APIs.

## Summary
- Named arguments pass values by parameter name with `name: value` syntax.
- They allow skipping optional parameters in the middle of a parameter list.
- Positional arguments must come before named arguments.
- Associative arrays can be unpacked as named arguments with `...$array`.
- Parameter names are now part of your public API contract.

## Code Examples

**Named arguments with constructor promotion and array unpacking for flexible object creation**

```php
<?php
declare(strict_types=1);

class Product {
    public function __construct(
        public string $name,
        public float $price,
        public int $stock = 0,
        public string $sku = '',
        public bool $active = true
    ) {}
}

// Named arguments make constructors self-documenting
$product = new Product(
    name: 'Widget',
    price: 29.99,
    active: false  // Skip stock and sku, use defaults
);

// Unpacking from an associative array
$data = ['name' => 'Gadget', 'price' => 49.99, 'stock' => 100];
$product2 = new Product(...$data);

echo $product->name;    // Widget
echo $product2->stock;  // 100
?>
```


## Resources

- [Named Arguments](https://www.php.net/manual/en/functions.arguments.php#functions.named-arguments) — Official documentation for named arguments

---

> 📘 *This lesson is part of the [Modern PHP 8.x: Latest Language Features](https://stanza.dev/courses/php-modern-features) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*