---
source_course: "php-essentials"
source_lesson: "php-essentials-defining-functions"
---

# Defining Functions in PHP

## Introduction
Functions are reusable blocks of code that perform specific tasks. They help you organize code, avoid repetition, and make programs easier to test and maintain. This lesson covers function syntax, parameters, return values, named arguments, and variadic functions.

## Key Concepts
- **Function**: A named, reusable block of code defined with the `function` keyword.
- **Parameter**: A variable declared in the function signature that receives a value when the function is called.
- **Return Value**: The result a function sends back to the caller using the `return` statement.
- **Named Arguments (PHP 8+)**: Allow calling a function by specifying parameter names, enabling you to skip optional parameters.
- **Variadic Parameters (`...$args`)**: Accept an unlimited number of arguments as an array.

## Real World Context
Functions are the building blocks of every PHP application. In frameworks like Laravel, every controller action, middleware check, and model method is a function. Well-designed functions with clear types and meaningful names make codebases maintainable and testable. Named arguments (PHP 8+) are especially valuable when working with functions that have many optional parameters.

## Deep Dive

### Basic Function Syntax

Define a function with the `function` keyword:

```php
<?php
function greet() {
    echo "Hello, World!";
}

greet();  // Outputs: Hello, World!
```

The function is defined once and can be called as many times as needed.

### Functions with Parameters

Parameters let you pass data into a function:

```php
<?php
function greet(string $name) {
    echo "Hello, $name!";
}

greet("Alice");  // Hello, Alice!
greet("Bob");    // Hello, Bob!
```

Adding the type declaration (`string`) ensures only the expected type is accepted.

### Default Parameter Values

Provide fallback values for optional parameters:

```php
<?php
function greet(string $name, string $greeting = "Hello") {
    echo "$greeting, $name!";
}

greet("Alice");           // Hello, Alice!
greet("Bob", "Welcome"); // Welcome, Bob!
```

Parameters with defaults must come after required parameters in the signature.

### Return Values

Use `return` to send a result back to the caller:

```php
<?php
function add(int $a, int $b): int {
    return $a + $b;
}

$sum = add(5, 3);
echo $sum;  // 8
```

The `: int` after the parameter list declares the return type.

### Returning Early

Return early to handle edge cases before the main logic:

```php
<?php
function divide(float $a, float $b): ?float {
    if ($b === 0.0) {
        return null;  // Early return for error case
    }
    return $a / $b;
}
```

The `?float` nullable return type allows the function to return `null` for the error case.

### Named Arguments (PHP 8+)

Call functions by naming the parameters explicitly:

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

// Skip $stock and name parameters explicitly:
createProduct(
    name: "Laptop",
    price: 999.99,
    featured: true  // Skip $stock, use its default
);
```

Named arguments make function calls readable and let you skip optional parameters.

### Variadic Functions

Accept an unlimited number of arguments using the spread operator:

```php
<?php
function sum(int ...$numbers): int {
    return array_sum($numbers);
}

echo sum(1, 2, 3);       // 6
echo sum(1, 2, 3, 4, 5); // 15
```

The `...$numbers` parameter collects all passed arguments into an array.

## Common Pitfalls
1. **Placing required parameters after optional ones** — `function test($a = 1, $b)` causes issues because you cannot skip `$a` without named arguments. Always put required parameters first.
2. **Forgetting the return statement** — A function without `return` (or with `return;`) returns `null`. If your function declares a return type, forgetting `return` causes a `TypeError`.
3. **Modifying arguments unintentionally** — PHP passes arguments by value by default. If you need to modify the original, use a reference parameter (`&$param`), but do so sparingly.

## Best Practices
1. **One function, one responsibility** — Each function should do exactly one thing. If you struggle to name it, it probably does too much.
2. **Always declare types** — Add parameter types and return types to every function. This serves as documentation and catches bugs early.
3. **Use named arguments for clarity** — When a function has more than 2-3 parameters or boolean flags, named arguments make the call site self-documenting.

## Summary
- Functions are defined with `function name() {}` and called with `name()`.
- Parameters accept data; return values send results back.
- Default parameter values make arguments optional.
- Named arguments (PHP 8+) allow skipping optional parameters and improve readability.
- Variadic parameters (`...$args`) accept unlimited arguments as an array.

## Code Examples

**A practical function with type declarations and named arguments — notice how discountPercent is named for clarity**

```php
<?php
declare(strict_types=1);

// A well-structured function with types and named arguments
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

// Using the function with named arguments
$original = 100.00;
$final = calculateDiscount($original, discountPercent: 15.0);

echo "Original: \$$original\n";        // Original: $100
echo "After discount: \$$final\n";     // After discount: $85
?>
```


## Resources

- [PHP Functions](https://www.php.net/manual/en/language.functions.php) — Complete guide to PHP functions

---

> 📘 *This lesson is part of the [PHP Essentials](https://stanza.dev/courses/php-essentials) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*