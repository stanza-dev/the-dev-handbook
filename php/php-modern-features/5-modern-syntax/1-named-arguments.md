---
source_course: "php-modern-features"
source_lesson: "php-modern-features-named-arguments"
---

# Named Arguments (PHP 8.0+)

Named arguments let you pass values by parameter name instead of position, making code clearer and more flexible.

## Basic Syntax

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

## Skipping Optional Parameters

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

## Mixing Positional and Named

```php
<?php
// Positional first, then named
createUser('John', 'john@example.com', age: 25, active: true);

// Cannot use positional after named!
createUser(name: 'John', 'john@example.com');  // Error!
```

## With Array Functions

```php
<?php
$numbers = [3, 1, 4, 1, 5, 9];

// Old way - what does 'true' mean?
array_filter($numbers, fn($n) => $n > 2, ARRAY_FILTER_USE_KEY);

// Named - self-documenting
array_filter(
    array: $numbers,
    callback: fn($n) => $n > 2,
    mode: ARRAY_FILTER_USE_BOTH
);
```

## With Constructors

```php
<?php
class Product {
    public function __construct(
        public string $name,
        public float $price,
        public int $stock = 0,
        public string $sku = '',
        public bool $active = true
    ) {}
}

$product = new Product(
    name: 'Widget',
    price: 29.99,
    active: false  // Skip stock and sku
);
```

## Argument Unpacking with Names

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

## Resources

- [Named Arguments](https://www.php.net/manual/en/functions.arguments.php#functions.named-arguments) — Official documentation for named arguments

---

> 📘 *This lesson is part of the [Modern PHP 8.x: Latest Language Features](https://stanza.dev/courses/php-modern-features) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*