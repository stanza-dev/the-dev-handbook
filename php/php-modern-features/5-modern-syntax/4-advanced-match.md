---
source_course: "php-modern-features"
source_lesson: "php-modern-features-advanced-match"
---

# Advanced Match & Pipe Operator

## Introduction
This lesson covers advanced match expression patterns and introduces PHP 8.5's pipe operator (`|>`), which enables left-to-right function chaining. Together, these features make PHP code more expressive and functional.

## Key Concepts
- **`match(true)` with Complex Conditions**: Using boolean expressions as match arms for range-based or multi-condition branching.
- **Pipe Operator (`|>`)**: PHP 8.5 syntax that passes the result of the left expression as the first argument to the right function.
- **Function Chaining**: Composing multiple transformations in a readable left-to-right pipeline.

## Real World Context
The pipe operator is one of the most requested PHP features, inspired by F#, Elixir, and the Unix pipe. It transforms deeply nested function calls like `strtolower(trim(strip_tags($input)))` into readable pipelines. Combined with match expressions, it enables clean data transformation code.

## Deep Dive

### Match with Complex Return Values

Match arms can return complex structures:

```php
<?php
enum HttpMethod { case GET; case POST; case PUT; case DELETE; }

function getRouteConfig(HttpMethod $method, string $resource): array {
    return match([$method, $resource]) {
        [HttpMethod::GET, 'users'] => [
            'controller' => 'UserController',
            'action' => 'index',
        ],
        [HttpMethod::POST, 'users'] => [
            'controller' => 'UserController',
            'action' => 'store',
        ],
        default => throw new NotFoundException(),
    };
}
```

Matching on arrays lets you branch on multiple values simultaneously.

### Nested Match Expressions

Match expressions can be nested:

```php
<?php
function getPricing(string $plan, bool $annual): array {
    return match($plan) {
        'basic' => [
            'name' => 'Basic',
            'price' => match($annual) {
                true => 99,
                false => 12,
            },
        ],
        'pro' => [
            'name' => 'Professional',
            'price' => match($annual) {
                true => 299,
                false => 35,
            },
        ],
        default => throw new InvalidArgumentException("Unknown plan: $plan"),
    };
}
```

The inner match selects the price based on billing period, while the outer match selects the plan.

### Match with Callbacks

Return closures from match for strategy patterns:

```php
<?php
$operation = 'uppercase';
$transformer = match($operation) {
    'uppercase' => fn($s) => strtoupper($s),
    'lowercase' => fn($s) => strtolower($s),
    'reverse' => fn($s) => strrev($s),
    'trim' => fn($s) => trim($s),
    default => fn($s) => $s,
};

echo $transformer('Hello');  // 'HELLO'
```

This pattern selects a transformation strategy at runtime.

### The Pipe Operator `|>` (PHP 8.5)

The pipe operator passes the left-side value as the first argument to the right-side function:

```php
<?php
// Before: nested calls (read inside-out)
$result = strtolower(trim(strip_tags($input)));

// After: pipe operator (read left-to-right)
$result = $input
    |> strip_tags(...)
    |> trim(...)
    |> strtolower(...);
```

The `(...)` syntax creates a closure from the function. The pipe operator passes the previous result as the first argument to each function in the chain.

### Pipe with Custom Functions

The pipe operator works with any callable:

```php
<?php
function addPrefix(string $s): string {
    return 'PREFIX_' . $s;
}

function truncate(string $s, int $maxLen = 50): string {
    return substr($s, 0, $maxLen);
}

$result = "  Hello World  "
    |> trim(...)
    |> strtoupper(...)
    |> addPrefix(...);
// 'PREFIX_HELLO WORLD'
```

Each step receives the output of the previous step as its first argument. Additional arguments use their default values.

### Pipe vs Method Chaining

The pipe operator brings functional composition to regular functions — no fluent builder classes needed:

```php
<?php
// Method chaining requires a builder class
$result = (new StringProcessor($input))
    ->stripTags()
    ->trim()
    ->toLower()
    ->toString();

// Pipe operator works with plain functions
$result = $input
    |> strip_tags(...)
    |> trim(...)
    |> strtolower(...);
```

The pipe version is simpler and works with any existing function without wrapper classes.

## Common Pitfalls
1. **Forgetting the `(...)` syntax** — The pipe operator requires a callable on the right side. Write `trim(...)` not `trim` to create a first-class callable.
2. **Multi-argument functions in pipes** — The pipe only passes one value (as the first argument). For functions needing more arguments, wrap them in a closure: `|> fn($x) => substr($x, 0, 10)`.

## Best Practices
1. **Use pipes for data transformation chains** — String processing, data cleaning, and formatting pipelines are ideal use cases for the pipe operator.
2. **Keep pipe chains short** — A pipeline of 3-5 steps is readable. Beyond that, extract helper functions to keep each step meaningful.

## Summary
- Advanced match patterns include array matching, nesting, and callback returns.
- PHP 8.5's pipe operator `|>` chains functions left-to-right.
- The pipe passes the left value as the first argument to the right function.
- Use `(...)` to create first-class callables for pipe chains.
- Pipes eliminate deeply nested function calls and reduce the need for fluent builders.

## Code Examples

**PHP 8.5 pipe operator for clean string transformation pipelines, including a URL slug generator**

```php
<?php
declare(strict_types=1);

// PHP 8.5 pipe operator for data processing
function sanitize(string $input): string {
    return $input
        |> strip_tags(...)
        |> trim(...)
        |> strtolower(...);
}

function formatSlug(string $title): string {
    return $title
        |> strip_tags(...)
        |> trim(...)
        |> strtolower(...)
        |> (fn($s) => preg_replace('/[^a-z0-9]+/', '-', $s))
        |> (fn($s) => trim($s, '-'));
}

echo sanitize('  <b>Hello World</b>  ');  // 'hello world'
echo formatSlug('  My Blog Post Title! ');  // 'my-blog-post-title'
?>
```


## Resources

- [Match Expression](https://www.php.net/manual/en/control-structures.match.php) — PHP match expression documentation

---

> 📘 *This lesson is part of the [Modern PHP 8.x: Latest Language Features](https://stanza.dev/courses/php-modern-features) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*