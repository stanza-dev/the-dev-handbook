---
source_course: "php-modern-features"
source_lesson: "php-modern-features-backed-enums"
---

# Backed Enums

## Introduction
Backed enums associate each case with a scalar value — either a `string` or an `int`. This is essential for database storage, API serialization, and interoperability with external systems that work with primitive values.

## Key Concepts
- **Backed Enum**: An enum where every case has an associated scalar value, declared with `: string` or `: int` after the enum name.
- **`value` Property**: Returns the scalar value associated with an enum case.
- **`from()` / `tryFrom()`**: Static methods to create an enum case from a scalar value.

## Real World Context
Databases store strings and integers, not PHP objects. When you save an order status to a `VARCHAR` column or receive a priority level from a JSON API, backed enums provide the bridge between scalar values and type-safe enum cases.

## Deep Dive

### String-Backed Enums

Declare the backing type after the enum name:

```php
<?php
enum Status: string {
    case Pending = 'pending';
    case Active = 'active';
    case Suspended = 'suspended';
    case Deleted = 'deleted';
}

$status = Status::Active;
echo $status->value;  // 'active'
echo $status->name;   // 'Active'
```

The `value` property returns the scalar, while `name` returns the case name. Both are strings but serve different purposes.

### Integer-Backed Enums

Integer backing is useful for HTTP status codes, priority levels, etc.:

```php
<?php
enum HttpStatus: int {
    case OK = 200;
    case Created = 201;
    case BadRequest = 400;
    case NotFound = 404;
    case ServerError = 500;
}

http_response_code(HttpStatus::NotFound->value);  // 404
```

The `->value` property gives you the integer for use with functions that expect primitives.

### Creating from Value: `from()` and `tryFrom()`

Convert scalar values back to enum cases:

```php
<?php
enum Status: string {
    case Pending = 'pending';
    case Active = 'active';
}

$status = Status::from('active');       // Status::Active
$status = Status::from('invalid');      // ValueError!

$status = Status::tryFrom('invalid');   // null (safe)
$status = Status::tryFrom('active');    // Status::Active
```

Use `from()` when the value must be valid (throws on failure). Use `tryFrom()` when the value might be invalid and you want to handle it gracefully.

### Database Integration

Backed enums integrate naturally with database operations:

```php
<?php
enum OrderStatus: string {
    case Pending = 'pending';
    case Processing = 'processing';
    case Shipped = 'shipped';
    case Delivered = 'delivered';
}

// Saving
$stmt = $pdo->prepare('UPDATE orders SET status = ? WHERE id = ?');
$stmt->execute([$order->status->value, $order->id]);

// Loading
$row = $stmt->fetch();
$status = OrderStatus::from($row['status']);
```

The `->value` property provides the string for storage, and `from()` converts it back.

### JSON Serialization

Combine backed enums with `JsonSerializable`:

```php
<?php
enum Priority: int {
    case Low = 1;
    case Medium = 2;
    case High = 3;
}

class Task implements JsonSerializable {
    public function __construct(
        public string $title,
        public Priority $priority
    ) {}
    
    public function jsonSerialize(): array {
        return [
            'title' => $this->title,
            'priority' => $this->priority->value,
        ];
    }
}

$task = new Task('Fix bug', Priority::High);
echo json_encode($task);
// {"title":"Fix bug","priority":3}
```

Use `->value` in serialization to output the scalar. Use `from()` or `tryFrom()` when deserializing.

## Common Pitfalls
1. **Forgetting to assign values to all cases** — In a backed enum, every case must have an explicit value. Unlike C-style enums, PHP does not auto-increment integer values.
2. **Using `from()` with untrusted input** — `from()` throws a `ValueError` on invalid input. Always use `tryFrom()` for user-supplied data and handle the `null` case.

## Best Practices
1. **Use `tryFrom()` at system boundaries** — When reading from databases, APIs, or user input, use `tryFrom()` and handle invalid values gracefully.
2. **Store backed enum values, not names** — Store `$enum->value` in databases, not `$enum->name`. The value is the stable contract; the name is a PHP-internal label that could change.

## Summary
- Backed enums associate each case with a `string` or `int` scalar value.
- Access the scalar with `->value` and the case name with `->name`.
- `from()` throws on invalid values; `tryFrom()` returns null.
- Use backed enums for database storage, JSON serialization, and API interop.
- Always use `tryFrom()` for untrusted input.

## Code Examples

**Payment status enum with state machine logic, demonstrating backed values and transition validation**

```php
<?php
declare(strict_types=1);

enum PaymentStatus: string {
    case Pending = 'pending';
    case Processing = 'processing';
    case Completed = 'completed';
    case Failed = 'failed';
    case Refunded = 'refunded';
    
    public function label(): string {
        return match($this) {
            self::Pending => 'Pending',
            self::Processing => 'Processing',
            self::Completed => 'Completed',
            self::Failed => 'Failed',
            self::Refunded => 'Refunded',
        };
    }
    
    public function isFinal(): bool {
        return in_array($this, [self::Completed, self::Failed, self::Refunded]);
    }
    
    public function canTransitionTo(self $newStatus): bool {
        return match($this) {
            self::Pending => in_array($newStatus, [self::Processing, self::Failed]),
            self::Processing => in_array($newStatus, [self::Completed, self::Failed]),
            self::Completed => $newStatus === self::Refunded,
            default => false,
        };
    }
}

$status = PaymentStatus::Pending;
if ($status->canTransitionTo(PaymentStatus::Processing)) {
    $status = PaymentStatus::Processing;
}
?>
```


## Resources

- [Backed Enumerations](https://www.php.net/manual/en/language.enumerations.backed.php) — Documentation for backed enums

---

> 📘 *This lesson is part of the [Modern PHP 8.x: Latest Language Features](https://stanza.dev/courses/php-modern-features) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*