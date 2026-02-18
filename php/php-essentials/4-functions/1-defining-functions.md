---
source_course: "php-essentials"
source_lesson: "php-essentials-defining-functions"
---

# Defining Functions in PHP

Functions are reusable blocks of code that perform specific tasks. They help you organize code and avoid repetition.

## Basic Function Syntax

```php
<?php
function greet() {
    echo "Hello, World!";
}

// Calling the function
greet();  // Outputs: Hello, World!
```

## Functions with Parameters

```php
<?php
function greet(string $name) {
    echo "Hello, $name!";
}

greet("Alice");  // Hello, Alice!
greet("Bob");    // Hello, Bob!
```

## Multiple Parameters

```php
<?php
function createUser(string $name, string $email, int $age) {
    echo "Creating user: $name ($email), Age: $age";
}

createUser("John", "john@example.com", 25);
```

## Default Parameter Values

```php
<?php
function greet(string $name, string $greeting = "Hello") {
    echo "$greeting, $name!";
}

greet("Alice");           // Hello, Alice!
greet("Bob", "Welcome"); // Welcome, Bob!
```

## Return Values

```php
<?php
function add(int $a, int $b): int {
    return $a + $b;
}

$sum = add(5, 3);
echo $sum;  // 8
```

## Returning Early

```php
<?php
function divide(float $a, float $b): ?float {
    if ($b === 0.0) {
        return null;  // Early return for error case
    }
    return $a / $b;
}
```

## Named Arguments (PHP 8+)

```php
<?php
function createProduct(
    string $name,
    float $price,
    int $stock = 0,
    bool $featured = false
) {
    // ...
}

// Skip parameters and name them:
createProduct(
    name: "Laptop",
    price: 999.99,
    featured: true  // Skip $stock, use default
);
```

## Variadic Functions

Accept unlimited arguments:

```php
<?php
function sum(int ...$numbers): int {
    return array_sum($numbers);
}

echo sum(1, 2, 3);       // 6
echo sum(1, 2, 3, 4, 5); // 15
```

## Code Examples

**A practical function with named arguments and types**

```php
<?php
declare(strict_types=1);

// A well-structured function with types
function calculateDiscount(
    float $price,
    float $discountPercent = 10.0,
    float $maxDiscount = 50.0
): float {
    $discount = $price * ($discountPercent / 100);
    
    // Cap the discount
    if ($discount > $maxDiscount) {
        $discount = $maxDiscount;
    }
    
    return $price - $discount;
}

// Using the function
$original = 100.00;
$final = calculateDiscount($original, discountPercent: 15.0);

echo "Original: \$$original\n";
echo "After discount: \$$final\n";
?>
```


## Resources

- [PHP Functions](https://www.php.net/manual/en/language.functions.php) — Complete guide to PHP functions

---

> 📘 *This lesson is part of the [PHP Essentials](https://stanza.dev/courses/php-essentials) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*