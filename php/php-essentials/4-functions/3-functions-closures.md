---
source_course: "php-essentials"
source_lesson: "php-essentials-anonymous-functions-closures"
---

# Anonymous Functions & Closures

## Introduction
Anonymous functions (also called closures) are functions without a name. They can be assigned to variables, passed as arguments, and even capture variables from their surrounding scope. PHP 7.4+ also introduced arrow functions for concise one-liners. This lesson covers both forms and when to use each.

## Key Concepts
- **Anonymous Function**: A function defined without a name, often assigned to a variable or passed as a callback.
- **Closure**: An anonymous function that captures variables from its surrounding scope using the `use` keyword.
- **Arrow Function (`fn() =>`)**: A short syntax for single-expression closures that automatically captures variables from the parent scope (PHP 7.4+).
- **First-Class Callable Syntax (`...')**: Creates a closure from any callable (PHP 8.1+).

## Real World Context
Anonymous functions and closures are essential for working with PHP's array functions (`array_map`, `array_filter`, `usort`), event listeners, middleware stacks, and dependency injection containers. Laravel uses closures extensively for route definitions, middleware, and query scopes. Arrow functions make these patterns even more concise.

## Deep Dive

### Basic Anonymous Function

Assign a function to a variable and call it:

```php
<?php
$greet = function(string $name): string {
    return "Hello, $name!";
};

echo $greet("Alice");  // Hello, Alice!
```

The variable `$greet` holds a `Closure` object that can be called like a regular function.

### As Callbacks

Anonymous functions shine when used as callbacks for array operations:

```php
<?php
$numbers = [1, 2, 3, 4, 5];

// Double each number
$doubled = array_map(function($n) {
    return $n * 2;
}, $numbers);
// [2, 4, 6, 8, 10]

// Filter even numbers
$evens = array_filter($numbers, function($n) {
    return $n % 2 === 0;
});
// [2, 4]
```

These inline functions are defined exactly where they are needed.

### Capturing Variables with `use`

Closures capture outside variables explicitly with the `use` keyword:

```php
<?php
$multiplier = 3;

$multiply = function(int $n) use ($multiplier): int {
    return $n * $multiplier;
};

echo $multiply(5);  // 15
```

Without `use ($multiplier)`, the variable would not be accessible inside the function. The value is captured by value at the time the closure is created.

### Capturing by Reference

To modify the original variable, capture by reference with `&`:

```php
<?php
$total = 0;

$add = function(int $amount) use (&$total): void {
    $total += $amount;
};

$add(10);
$add(20);
echo $total;  // 30
```

The `&` allows the closure to modify `$total` in the outer scope.

### Arrow Functions (PHP 7.4+)

Arrow functions are a concise syntax for simple closures:

```php
<?php
// Traditional closure
$double = function($n) {
    return $n * 2;
};

// Arrow function (equivalent)
$double = fn($n) => $n * 2;

echo $double(5);  // 10
```

Arrow functions have two key differences from regular closures:

1. They automatically capture variables from the parent scope (no `use` needed)
2. They support only a single expression (the result is implicitly returned)

Automatic capture example:

```php
<?php
$factor = 3;

$multiply = fn($n) => $n * $factor;  // $factor captured automatically

$numbers = [1, 2, 3];
$result = array_map(fn($n) => $n * $factor, $numbers);
// [3, 6, 9]
```

Arrow functions always capture by value, never by reference.

### First-Class Callable Syntax (PHP 8.1+)

Create a closure from any existing function or method:

```php
<?php
$strlen = strlen(...);
echo $strlen("hello");  // 5

class Calculator {
    public function add(int $a, int $b): int {
        return $a + $b;
    }
}

$calc = new Calculator();
$adder = $calc->add(...);
echo $adder(2, 3);  // 5
```

The `...` syntax wraps the callable in a `Closure` object, making it passable as a callback.

## Common Pitfalls
1. **Forgetting `use` with regular closures** — Unlike arrow functions, traditional closures do not automatically capture outer variables. Omitting `use` means the variable is simply undefined inside the function.
2. **Expecting arrow functions to capture by reference** — Arrow functions always capture by value. If you need to modify an outer variable, use a traditional closure with `use (&$var)`.
3. **Using arrow functions for multi-statement logic** — Arrow functions support only a single expression. If you need multiple statements, use a regular anonymous function.

## Best Practices
1. **Use arrow functions for short callbacks** — `fn($x) => $x * 2` is cleaner than a full anonymous function for simple transformations.
2. **Use traditional closures for complex logic** — When you need multiple statements, explicit `use`, or reference capture, a traditional closure is the right choice.
3. **Prefer named functions for reused logic** — If you use the same closure in multiple places, extract it into a named function for clarity and reusability.

## Summary
- Anonymous functions are defined without a name and can be assigned to variables or passed as callbacks.
- Use `use ($var)` to capture outer variables in traditional closures; use `use (&$var)` for reference capture.
- Arrow functions (`fn() =>`) auto-capture variables and support a single expression with implicit return.
- First-class callable syntax (`strlen(...)`) wraps any callable as a Closure (PHP 8.1+).
- Arrow functions capture by value at creation time, not by reference.

## Code Examples

**Using arrow functions for data filtering and transformation — notice how $minPrice is captured automatically without the use keyword**

```php
<?php
// Real-world: Filtering and transforming data with arrow functions
$products = [
    ['name' => 'Laptop', 'price' => 999, 'category' => 'electronics'],
    ['name' => 'Shirt', 'price' => 29, 'category' => 'clothing'],
    ['name' => 'Phone', 'price' => 699, 'category' => 'electronics'],
    ['name' => 'Pants', 'price' => 49, 'category' => 'clothing'],
];

$minPrice = 50;

// Filter electronics over $minPrice using arrow functions
// Arrow functions auto-capture $minPrice from the outer scope
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