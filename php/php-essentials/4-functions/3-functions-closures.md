---
source_course: "php-essentials"
source_lesson: "php-essentials-anonymous-functions-closures"
---

# Anonymous Functions & Closures

Anonymous functions (also called closures) are functions without names. They're useful for callbacks and short-lived functionality.

## Basic Anonymous Function

```php
<?php
$greet = function(string $name): string {
    return "Hello, $name!";
};

echo $greet("Alice");  // Hello, Alice!
```

## As Callbacks

```php
<?php
$numbers = [1, 2, 3, 4, 5];

// Double each number
$doubled = array_map(function($n) {
    return $n * 2;
}, $numbers);

print_r($doubled);  // [2, 4, 6, 8, 10]

// Filter even numbers
$evens = array_filter($numbers, function($n) {
    return $n % 2 === 0;
});

print_r($evens);  // [2, 4]
```

## Capturing Variables with `use`

Closures can access outside variables:

```php
<?php
$multiplier = 3;

$multiply = function(int $n) use ($multiplier): int {
    return $n * $multiplier;
};

echo $multiply(5);  // 15
```

## Capturing by Reference

```php
<?php
$total = 0;

$add = function(int $amount) use (&$total): void {
    $total += $amount;  // Modifies the original
};

$add(10);
$add(20);
echo $total;  // 30
```

## Arrow Functions (PHP 7.4+)

Short syntax for simple closures:

```php
<?php
// Traditional closure
$double = function($n) {
    return $n * 2;
};

// Arrow function (same thing)
$double = fn($n) => $n * 2;

echo $double(5);  // 10
```

## Arrow Function Features

1. Automatically captures variables (no `use` needed)
2. Single expression only (implicit return)
3. Cannot have multiple statements

```php
<?php
$factor = 3;

// Arrow function auto-captures $factor
$multiply = fn($n) => $n * $factor;

$numbers = [1, 2, 3];
$result = array_map(fn($n) => $n * $factor, $numbers);
// [3, 6, 9]
```

## First-Class Callable Syntax (PHP 8.1+)

Create a closure from any callable:

```php
<?php
class Calculator {
    public function add(int $a, int $b): int {
        return $a + $b;
    }
}

$calc = new Calculator();
$adder = $calc->add(...);

echo $adder(2, 3);  // 5
```

## Code Examples

**Using arrow functions for data filtering and transformation**

```php
<?php
// Real-world: Filtering and transforming data
$products = [
    ['name' => 'Laptop', 'price' => 999, 'category' => 'electronics'],
    ['name' => 'Shirt', 'price' => 29, 'category' => 'clothing'],
    ['name' => 'Phone', 'price' => 699, 'category' => 'electronics'],
    ['name' => 'Pants', 'price' => 49, 'category' => 'clothing'],
];

$minPrice = 50;

// Filter electronics over $minPrice using arrow functions
$expensiveElectronics = array_filter(
    $products,
    fn($p) => $p['category'] === 'electronics' && $p['price'] >= $minPrice
);

// Get just the names
$names = array_map(fn($p) => $p['name'], $expensiveElectronics);

print_r($names);  // ['Laptop', 'Phone']
?>
```


## Resources

- [Anonymous Functions](https://www.php.net/manual/en/functions.anonymous.php) — Official guide to PHP closures and anonymous functions
- [Arrow Functions](https://www.php.net/manual/en/functions.arrow.php) — Documentation for PHP 7.4+ arrow functions

---

> 📘 *This lesson is part of the [PHP Essentials](https://stanza.dev/courses/php-essentials) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*