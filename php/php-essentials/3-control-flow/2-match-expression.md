---
source_course: "php-essentials"
source_lesson: "php-essentials-match-expression"
---

# The Match Expression

## Introduction
PHP 8 introduced the `match` expression as a more powerful, safer alternative to the traditional `switch` statement. It returns a value, uses strict comparison by default, and eliminates the need for `break` statements. This lesson explains when and how to use `match` effectively.

## Key Concepts
- **`match` Expression**: A control structure that maps input values to results using strict comparison (`===`).
- **Expression vs Statement**: `match` is an expression that returns a value, unlike `switch` which is a statement that executes blocks.
- **No Fall-Through**: Each arm in `match` is independent. There is no fall-through behavior and no need for `break`.
- **`UnhandledMatchError`**: If no arm matches and there is no `default`, PHP throws an `UnhandledMatchError`.

## Real World Context
The `match` expression is commonly used for mapping status codes to messages, routing actions to handlers, converting enum values, and any situation where you need to map one value to another. Its strict comparison and return-value semantics make it ideal for replacing verbose switch statements in modern PHP applications.

## Deep Dive

### Basic Match Syntax

A `match` expression maps input to output values:

```php
<?php
$status = 200;

$message = match($status) {
    200 => 'OK',
    201 => 'Created',
    400 => 'Bad Request',
    404 => 'Not Found',
    500 => 'Server Error',
};

echo $message;  // 'OK'
```

The result of the matching arm is returned and assigned to `$message`. No `break` needed.

### Match vs Switch

Here is a side-by-side comparison:

| Feature | `match` | `switch` |
|---------|---------|----------|
| Comparison | Strict (`===`) | Loose (`==`) |
| Returns value | Yes | No |
| Fall-through | No | Yes (without break) |
| Unmatched input | Throws error | Silently continues |

The strict comparison in `match` prevents subtle bugs from type coercion.

### Multiple Conditions

A single arm can match multiple values separated by commas:

```php
<?php
$day = 'Saturday';

$type = match($day) {
    'Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday' => 'Weekday',
    'Saturday', 'Sunday' => 'Weekend',
};

echo $type;  // 'Weekend'
```

This is equivalent to grouping `case` statements in a switch, but much cleaner.

### Default Case

Always include a `default` arm to handle unexpected values:

```php
<?php
$code = 418;

$message = match($code) {
    200 => 'OK',
    404 => 'Not Found',
    default => 'Unknown Status',
};

echo $message;  // 'Unknown Status'
```

Without `default`, an unmatched value throws `UnhandledMatchError`.

### Match with Complex Expressions

Use `match(true)` to match against boolean conditions, similar to an if-elseif chain:

```php
<?php
$age = 25;

$category = match(true) {
    $age < 13 => 'child',
    $age < 20 => 'teenager',
    $age < 65 => 'adult',
    default => 'senior',
};

echo $category;  // 'adult'
```

This pattern evaluates each condition in order and returns the first match, making it a concise alternative to long if-elseif chains.

## Common Pitfalls
1. **Forgetting the `default` arm** — Without `default`, an unmatched value throws `UnhandledMatchError` at runtime. Always include `default` unless you are certain all possible values are covered.
2. **Expecting fall-through behavior** — Unlike `switch`, `match` arms are independent. If you need fall-through, list multiple values on the same arm using commas.
3. **Using `match` for side effects** — `match` is designed to return values. If you need to execute multiple statements, use `switch` or if/else instead.

## Best Practices
1. **Prefer `match` over `switch` for value mapping** — When you are assigning a variable based on a condition, `match` is cleaner, safer, and more concise.
2. **Always include `default`** — Even when you think you have covered all cases, a `default` arm provides a safety net against unexpected input.
3. **Use `match(true)` for range conditions** — It is more readable than a chain of if-elseif statements when each condition is a simple boolean check.

## Summary
- `match` is an expression that returns a value and uses strict comparison.
- It does not have fall-through behavior; no `break` is needed.
- Use commas to match multiple values on a single arm.
- Always include a `default` arm to prevent `UnhandledMatchError`.
- `match(true)` enables range-based matching similar to if-elseif chains.

## Code Examples

**Using match to map HTTP status codes to categorized response objects — notice how multiple codes share a single arm**

```php
<?php
// HTTP status code handler using match
function getStatusInfo(int $code): array {
    return match($code) {
        200, 201, 204 => [
            'type' => 'success',
            'icon' => 'check'
        ],
        301, 302, 307 => [
            'type' => 'redirect',
            'icon' => 'arrow'
        ],
        400, 401, 403, 404 => [
            'type' => 'client_error',
            'icon' => 'warning'
        ],
        500, 502, 503 => [
            'type' => 'server_error',
            'icon' => 'error'
        ],
        default => [
            'type' => 'unknown',
            'icon' => 'question'
        ],
    };
}

print_r(getStatusInfo(404));
// ['type' => 'client_error', 'icon' => 'warning']
?>
```


## Resources

- [Match Expression](https://www.php.net/manual/en/control-structures.match.php) — Official documentation for PHP 8 match expression

---

> 📘 *This lesson is part of the [PHP Essentials](https://stanza.dev/courses/php-essentials) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*