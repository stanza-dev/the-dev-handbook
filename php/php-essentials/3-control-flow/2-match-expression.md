---
source_course: "php-essentials"
source_lesson: "php-essentials-match-expression"
---

# The Match Expression

PHP 8 introduced the `match` expression as a more powerful alternative to `switch`. It's more concise and safer.

## Basic Match Syntax

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

## Match vs Switch

| Feature | `match` | `switch` |
|---------|---------|----------|
| Comparison | Strict (`===`) | Loose (`==`) |
| Returns value | Yes | No |
| Fall-through | No | Yes (without break) |
| Default required | If no match, throws | Optional |
| Multiple conditions | Yes | Yes |

## Multiple Conditions

```php
<?php
$day = 'Saturday';

$type = match($day) {
    'Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday' => 'Weekday',
    'Saturday', 'Sunday' => 'Weekend',
};

echo $type;  // 'Weekend'
```

## Default Case

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

## Match with Complex Expressions

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

## When to Use Match

1. **Returning values** based on conditions
2. **Strict comparisons** needed
3. **Multiple values** mapping to same result
4. **No fall-through** behavior needed

## Error Handling

```php
<?php
$value = 'unknown';

// This throws UnhandledMatchError if no match!
$result = match($value) {
    'a' => 1,
    'b' => 2,
    // Missing default = UnhandledMatchError
};

// Always include default for safety
$result = match($value) {
    'a' => 1,
    'b' => 2,
    default => 0,
};
```

## Code Examples

**Using match for HTTP status code handling**

```php
<?php
// HTTP status code handler using match
function getStatusInfo(int $code): array {
    return match($code) {
        200, 201, 204 => [
            'type' => 'success',
            'icon' => '✓'
        ],
        301, 302, 307 => [
            'type' => 'redirect',
            'icon' => '→'
        ],
        400, 401, 403, 404 => [
            'type' => 'client_error',
            'icon' => '⚠'
        ],
        500, 502, 503 => [
            'type' => 'server_error',
            'icon' => '✗'
        ],
        default => [
            'type' => 'unknown',
            'icon' => '?'
        ],
    };
}

print_r(getStatusInfo(404));
// ['type' => 'client_error', 'icon' => '⚠']
?>
```


## Resources

- [Match Expression](https://www.php.net/manual/en/control-structures.match.php) — Official documentation for PHP 8 match expression

---

> 📘 *This lesson is part of the [PHP Essentials](https://stanza.dev/courses/php-essentials) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*