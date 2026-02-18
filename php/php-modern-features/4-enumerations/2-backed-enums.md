---
source_course: "php-modern-features"
source_lesson: "php-modern-features-backed-enums"
---

# Backed Enums

Backed enums associate each case with a scalar value (int or string). Essential for database storage and API responses.

## String-Backed Enums

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

## Integer-Backed Enums

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

## Creating from Value

```php
<?php
enum Status: string {
    case Pending = 'pending';
    case Active = 'active';
}

// From database value
$dbValue = 'active';
$status = Status::from($dbValue);  // Status::Active

// With invalid value
$status = Status::from('invalid');  // ValueError!

// Safe version - returns null if not found
$status = Status::tryFrom('invalid');  // null
$status = Status::tryFrom('active');   // Status::Active
```

## Database Integration

```php
<?php
enum OrderStatus: string {
    case Pending = 'pending';
    case Processing = 'processing';
    case Shipped = 'shipped';
    case Delivered = 'delivered';
    case Cancelled = 'cancelled';
}

class Order {
    public function __construct(
        public int $id,
        public OrderStatus $status
    ) {}
}

// Saving to database
$order = new Order(1, OrderStatus::Processing);
$stmt = $pdo->prepare('UPDATE orders SET status = ? WHERE id = ?');
$stmt->execute([$order->status->value, $order->id]);

// Loading from database
$row = $stmt->fetch();
$status = OrderStatus::from($row['status']);
```

## JSON Serialization

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

## Code Examples

**Payment status enum with state machine logic**

```php
<?php
declare(strict_types=1);

// Complete payment status enum with methods
enum PaymentStatus: string {
    case Pending = 'pending';
    case Processing = 'processing';
    case Completed = 'completed';
    case Failed = 'failed';
    case Refunded = 'refunded';
    
    public function label(): string {
        return match($this) {
            self::Pending => '⏳ Pending',
            self::Processing => '🔄 Processing',
            self::Completed => '✅ Completed',
            self::Failed => '❌ Failed',
            self::Refunded => '↩️ Refunded',
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

// Usage
$status = PaymentStatus::Pending;
echo $status->label();  // ⏳ Pending

if ($status->canTransitionTo(PaymentStatus::Processing)) {
    $status = PaymentStatus::Processing;
}
?>
```


## Resources

- [Backed Enumerations](https://www.php.net/manual/en/language.enumerations.backed.php) — Documentation for backed enums

---

> 📘 *This lesson is part of the [Modern PHP 8.x: Latest Language Features](https://stanza.dev/courses/php-modern-features) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*