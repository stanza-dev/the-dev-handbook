---
source_course: "php-modern-features"
source_lesson: "php-modern-features-match-expression-advanced"
---

# Match Expression Advanced Patterns

The match expression (PHP 8.0+) is a more powerful switch. Let's explore advanced usage patterns.

## Match Returns Values

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

## Multiple Values Per Arm

```php
<?php
$day = 'Saturday';

$type = match($day) {
    'Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday' => 'Weekday',
    'Saturday', 'Sunday' => 'Weekend',
};
```

## Matching with Expressions

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

## Complex Return Values

```php
<?php
$action = 'create';

[$method, $template] = match($action) {
    'create' => ['POST', 'form.html'],
    'edit' => ['PUT', 'form.html'],
    'delete' => ['DELETE', 'confirm.html'],
    'view' => ['GET', 'show.html'],
    default => throw new InvalidArgumentException("Unknown action: $action"),
};
```

## Throwing in Match

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

## Match vs Switch Comparison

```php
<?php
// Switch - loose comparison, falls through
switch ($value) {
    case 0:
    case '0':    // Both match due to loose comparison!
        $result = 'zero';
        break;
    default:
        $result = 'other';
}

// Match - strict comparison, no fall-through
$result = match($value) {
    0 => 'integer zero',
    '0' => 'string zero',
    default => 'other',
};
```

## Enum with Match

```php
<?php
enum Status {
    case Active;
    case Inactive;
    case Pending;
}

$status = Status::Active;

$color = match($status) {
    Status::Active => 'green',
    Status::Inactive => 'gray',
    Status::Pending => 'yellow',
};
```

## Code Examples

**HTTP response builder using nested match expressions**

```php
<?php
declare(strict_types=1);

// HTTP Response builder using match
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
                    422 => 'Validation Error',
                    default => 'Client Error',
                },
            ],
            $code >= 500 => [
                'type' => 'server_error',
                'message' => match($code) {
                    500 => 'Internal Server Error',
                    502 => 'Bad Gateway',
                    503 => 'Service Unavailable',
                    default => 'Server Error',
                },
            ],
            default => [
                'type' => 'unknown',
                'message' => 'Unknown Status',
            ],
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