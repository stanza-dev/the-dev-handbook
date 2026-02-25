---
source_course: "php-modern-features"
source_lesson: "php-modern-features-match-expression-advanced"
---

# Match Expression Patterns

## Introduction
The `match` expression, introduced in PHP 8.0, is a more powerful and safer alternative to `switch`. It uses strict comparison, returns a value, has no fall-through, and throws an error when no arm matches. This lesson covers both basic and advanced usage patterns.

## Key Concepts
- **Match Expression**: A compact alternative to `switch` that uses strict (`===`) comparison and returns a value.
- **No Fall-Through**: Each arm is independent — no `break` statements needed.
- **`match(true)` Pattern**: Using `true` as the subject to evaluate boolean expressions in each arm.

## Real World Context
The `switch` statement has been a source of bugs for decades: loose comparison (`==`), accidental fall-through, and no return value. The `match` expression eliminates all three problems. It is now the recommended way to branch on values in modern PHP.

## Deep Dive

### Match Returns Values

Unlike `switch`, `match` is an expression that returns a value:

```php
<?php
$status = 'active';

$message = match($status) {
    'active' => 'User is currently active',
    'pending' => 'Awaiting activation',
    'banned' => 'User has been banned',
    default => 'Unknown status',
};

echo $message;
```

The result is assigned directly to a variable — no temporary variable or `break` needed.

### Multiple Values Per Arm

Group multiple values with commas:

```php
<?php
$day = 'Saturday';

$type = match($day) {
    'Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday' => 'Weekday',
    'Saturday', 'Sunday' => 'Weekend',
};
```

This is cleaner than `switch` with fall-through cases.

### Matching with Expressions: `match(true)`

Use `match(true)` to evaluate conditions:

```php
<?php
$age = 25;

$category = match(true) {
    $age < 13 => 'child',
    $age < 20 => 'teenager',
    $age < 30 => 'young adult',
    $age < 60 => 'adult',
    default => 'senior',
};
```

Each arm is a boolean expression. The first one that evaluates to `true` wins.

### Throwing in Match Arms

Throw exceptions directly:

```php
<?php
function getStatusCode(string $status): int {
    return match($status) {
        'ok' => 200,
        'created' => 201,
        'not_found' => 404,
        'error' => 500,
        default => throw new ValueError("Invalid status: $status"),
    };
}
```

The `throw` expression (PHP 8.0) works seamlessly in match arms.

### Match vs Switch: Strict Comparison

This is the most important behavioral difference:

```php
<?php
// Switch uses loose comparison (==)
switch ($value) {
    case 0:
    case '0':    // Both match 0 due to loose comparison!
        $result = 'zero';
        break;
}

// Match uses strict comparison (===)
$result = match($value) {
    0 => 'integer zero',
    '0' => 'string zero',  // Different from integer 0
    default => 'other',
};
```

With `match`, `0` and `'0'` are different values. This prevents the most common class of `switch` bugs.

### Match with Enums

Enums and match are a natural pairing:

```php
<?php
enum Status { case Active; case Inactive; case Pending; }

$color = match($status) {
    Status::Active => 'green',
    Status::Inactive => 'gray',
    Status::Pending => 'yellow',
};
```

If you omit a case and there is no `default`, PHP throws an `UnhandledMatchError`, ensuring exhaustive handling.

## Common Pitfalls
1. **Missing `default` without exhaustive arms** — If no arm matches and there is no `default`, PHP throws an `UnhandledMatchError`. This is actually a feature for enums (ensures exhaustiveness), but can be surprising for string inputs.
2. **Expecting fall-through** — `match` has no fall-through. Use comma-separated values to group cases, not sequential arms.

## Best Practices
1. **Use `match` instead of `switch` by default** — Strict comparison and no fall-through make `match` safer. Only use `switch` if you need fall-through or statement bodies.
2. **Omit `default` for enums** — Without a `default`, adding a new enum case forces you to handle it everywhere `match` is used, preventing missed cases.

## Summary
- `match` is an expression: it returns a value and uses strict comparison.
- No fall-through — each arm is independent.
- Use `match(true)` for condition-based branching.
- Throw expressions work inside match arms.
- Omit `default` for enums to get compile-time exhaustiveness checking.

## Code Examples

**HTTP response builder using nested match expressions for clean status code mapping**

```php
<?php
declare(strict_types=1);

class ResponseBuilder {
    public static function fromStatusCode(int $code): array {
        return match(true) {
            $code >= 200 && $code < 300 => [
                'type' => 'success',
                'message' => match($code) {
                    200 => 'OK',
                    201 => 'Created',
                    204 => 'No Content',
                    default => 'Success',
                },
            ],
            $code >= 400 && $code < 500 => [
                'type' => 'client_error',
                'message' => match($code) {
                    400 => 'Bad Request',
                    401 => 'Unauthorized',
                    403 => 'Forbidden',
                    404 => 'Not Found',
                    default => 'Client Error',
                },
            ],
            $code >= 500 => [
                'type' => 'server_error',
                'message' => match($code) {
                    500 => 'Internal Server Error',
                    503 => 'Service Unavailable',
                    default => 'Server Error',
                },
            ],
            default => ['type' => 'unknown', 'message' => 'Unknown Status'],
        };
    }
}

print_r(ResponseBuilder::fromStatusCode(404));
// ['type' => 'client_error', 'message' => 'Not Found']
?>
```


## Resources

- [Match Expression](https://www.php.net/manual/en/control-structures.match.php) — Complete match expression documentation

---

> 📘 *This lesson is part of the [Modern PHP 8.x: Latest Language Features](https://stanza.dev/courses/php-modern-features) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*