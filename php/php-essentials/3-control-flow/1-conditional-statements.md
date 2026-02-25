---
source_course: "php-essentials"
source_lesson: "php-essentials-conditional-statements"
---

# Conditional Statements

## Introduction
Conditional statements allow your code to make decisions and execute different blocks based on conditions. They are the foundation of any program's logic. This lesson covers `if/else`, the ternary operator, and PHP's powerful null coalescing operators.

## Key Concepts
- **`if/elseif/else`**: The primary conditional structure. Tests conditions sequentially and executes the first matching block.
- **Ternary Operator (`? :`)**: A shorthand for simple if-else expressions. Returns one of two values based on a condition.
- **Null Coalescing Operator (`??`)**: Returns the first value that exists and is not null. Eliminates verbose `isset()` checks.
- **Strict vs Loose Comparison**: `===` checks both value and type; `==` checks value only with type coercion.

## Real World Context
Every web application uses conditionals for authentication ("is the user logged in?"), authorization ("does this user have admin privileges?"), input validation ("is the email format valid?"), and rendering logic ("show the dashboard or the login page?"). PHP's null coalescing operator is especially useful when handling optional query parameters and configuration values.

## Deep Dive

### The `if` Statement

The simplest conditional executes a block when a condition is true:

```php
<?php
$age = 18;

if ($age >= 18) {
    echo "You are an adult.";
}
```

The condition inside the parentheses is evaluated as a boolean. If it is true, the block executes.

### `if-else` Statement

Handle two possible outcomes:

```php
<?php
$score = 75;

if ($score >= 60) {
    echo "You passed!";
} else {
    echo "You failed.";
}
```

### `if-elseif-else` Chain

Handle multiple conditions by chaining `elseif` blocks:

```php
<?php
$grade = 85;

if ($grade >= 90) {
    echo "A - Excellent!";
} elseif ($grade >= 80) {
    echo "B - Good job!";
} elseif ($grade >= 70) {
    echo "C - Satisfactory";
} elseif ($grade >= 60) {
    echo "D - Needs improvement";
} else {
    echo "F - Failed";
}
```

PHP evaluates conditions top-to-bottom and executes only the first matching block.

### Ternary Operator

A concise shorthand for simple if-else assignments:

```php
<?php
$age = 20;
$status = $age >= 18 ? "adult" : "minor";

// Equivalent to:
if ($age >= 18) {
    $status = "adult";
} else {
    $status = "minor";
}
```

The syntax is `condition ? value_if_true : value_if_false`. Use it for simple expressions; avoid nesting ternaries as they become unreadable.

### Null Coalescing Operator (`??`)

Return the first non-null value, perfect for providing defaults:

```php
<?php
$username = $_GET['user'] ?? 'Guest';
// If $_GET['user'] is null or undefined, use 'Guest'

// Chain multiple fallbacks:
$name = $firstName ?? $nickname ?? 'Anonymous';
```

Unlike the ternary, `??` does not trigger a notice for undefined variables.

### Null Coalescing Assignment (`??=`)

Assign a value only if the variable is currently null (PHP 7.4+):

```php
<?php
$config['timeout'] ??= 30;
// Sets to 30 only if $config['timeout'] is null or undefined
```

### Comparison Operators

A complete reference for PHP comparisons:

| Operator | Description | Example |
|----------|-------------|--------|
| `==` | Equal value | `5 == "5"` → true |
| `===` | Identical (value & type) | `5 === "5"` → false |
| `!=` | Not equal | `5 != 3` → true |
| `!==` | Not identical | `5 !== "5"` → true |
| `<=>` | Spaceship | `1 <=> 2` → -1 |

The spaceship operator (`<=>`) returns -1, 0, or 1 and is useful for custom sorting.

## Common Pitfalls
1. **Using `==` instead of `===`** — Loose comparison can produce surprising results: `0 == "foo"` is `true` because the string is coerced to `0`. Always prefer strict comparison (`===`).
2. **Nesting ternary operators** — `$a ? $b : $c ? $d : $e` is confusing and its behavior changed in PHP 8. Use explicit `if/else` for complex logic.
3. **Confusing `??` with `?:`** — The null coalescing operator (`??`) only checks for null. The short ternary (`?:`) checks for any falsy value. `"" ?? "default"` returns `""`, but `"" ?: "default"` returns `"default"`.

## Best Practices
1. **Use `===` by default** — Strict comparison prevents type coercion bugs. Only use `==` when you intentionally want loose comparison.
2. **Use `??` for defaults** — Replace `isset($x) ? $x : 'default'` with the cleaner `$x ?? 'default'`.
3. **Keep conditions simple** — Extract complex conditions into well-named boolean variables: `$isEligible = $age >= 18 && $hasConsent;`

## Summary
- `if/elseif/else` is the primary conditional structure in PHP.
- The ternary operator (`? :`) is a concise shorthand for simple conditionals.
- The null coalescing operator (`??`) provides elegant default values for null or undefined variables.
- Always prefer strict comparison (`===`) over loose comparison (`==`).

## Code Examples

**Practical conditional logic for user authentication — combines if/elseif/else with null coalescing and ternary operators**

```php
<?php
// Real-world example: User authentication and authorization
$user = $_SESSION['user'] ?? null;
$role = $user['role'] ?? 'guest';

if ($role === 'admin') {
    echo "Welcome, Administrator!";
    // Show admin dashboard
} elseif ($role === 'member') {
    echo "Welcome back, Member!";
    // Show member area
} else {
    echo "Please log in to continue.";
    // Show login form
}

// Using ternary for simple display logic
$greeting = $user ? "Hello, {$user['name']}" : "Hello, Guest";
echo $greeting;
?>
```


## Resources

- [PHP Control Structures](https://www.php.net/manual/en/language.control-structures.php) — Official guide to all PHP control structures

---

> 📘 *This lesson is part of the [PHP Essentials](https://stanza.dev/courses/php-essentials) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*