---
source_course: "php-oop-mastery"
source_lesson: "php-oop-mastery-solid-principles"
---

# SOLID Principles

Five design principles for maintainable, flexible object-oriented code.

## S - Single Responsibility Principle

**A class should have only one reason to change.**

```php
<?php
// BAD: Multiple responsibilities
class UserManager {
    public function createUser(array $data): User { /* ... */ }
    public function sendWelcomeEmail(User $user): void { /* ... */ }
    public function generateReport(): string { /* ... */ }
}

// GOOD: Single responsibility each
class UserRepository {
    public function create(array $data): User { /* ... */ }
}

class WelcomeEmailSender {
    public function send(User $user): void { /* ... */ }
}

class UserReportGenerator {
    public function generate(): string { /* ... */ }
}
```

## O - Open/Closed Principle

**Open for extension, closed for modification.**

```php
<?php
// BAD: Must modify to add new types
class PaymentProcessor {
    public function process(string $type, float $amount): void {
        if ($type === 'credit') {
            // Process credit
        } elseif ($type === 'paypal') {
            // Process PayPal
        }
        // Must add more elseif for new types!
    }
}

// GOOD: Extend without modifying
interface PaymentGateway {
    public function process(float $amount): void;
}

class CreditCardGateway implements PaymentGateway {
    public function process(float $amount): void { /* ... */ }
}

class PayPalGateway implements PaymentGateway {
    public function process(float $amount): void { /* ... */ }
}

// Add new payment types without changing existing code
class CryptoGateway implements PaymentGateway {
    public function process(float $amount): void { /* ... */ }
}
```

## L - Liskov Substitution Principle

**Subtypes must be substitutable for their base types.**

```php
<?php
// BAD: Square violates LSP
class Rectangle {
    protected int $width;
    protected int $height;
    
    public function setWidth(int $width): void {
        $this->width = $width;
    }
    
    public function setHeight(int $height): void {
        $this->height = $height;
    }
    
    public function getArea(): int {
        return $this->width * $this->height;
    }
}

class Square extends Rectangle {
    public function setWidth(int $width): void {
        $this->width = $width;
        $this->height = $width;  // Breaks expectations!
    }
}

// Code expecting Rectangle behavior will fail with Square
```

## I - Interface Segregation Principle

**Clients shouldn't depend on interfaces they don't use.**

```php
<?php
// BAD: Fat interface
interface Worker {
    public function work(): void;
    public function eat(): void;
    public function sleep(): void;
}

// Robots can't eat or sleep!
class Robot implements Worker { /* ... */ }

// GOOD: Segregated interfaces
interface Workable {
    public function work(): void;
}

interface Eatable {
    public function eat(): void;
}

class Human implements Workable, Eatable { /* ... */ }
class Robot implements Workable { /* ... */ }  // Only what it needs
```

## D - Dependency Inversion Principle

**Depend on abstractions, not concretions.**

```php
<?php
// BAD: Depends on concrete class
class OrderService {
    private MySQLDatabase $db;  // Concrete!
    
    public function __construct() {
        $this->db = new MySQLDatabase();
    }
}

// GOOD: Depends on abstraction
interface DatabaseInterface {
    public function query(string $sql): array;
}

class OrderService {
    public function __construct(
        private DatabaseInterface $db  // Abstract!
    ) {}
}

// Can inject any implementation
$service = new OrderService(new MySQLDatabase());
$service = new OrderService(new PostgreSQLDatabase());
$service = new OrderService(new InMemoryDatabase());  // For tests
```

## Code Examples

**SOLID-compliant notification system**

```php
<?php
declare(strict_types=1);

// SOLID-compliant notification system

// Single Responsibility: Each class does one thing
interface NotificationChannel {
    public function send(string $recipient, string $message): void;
}

class EmailChannel implements NotificationChannel {
    public function send(string $recipient, string $message): void {
        // Send email
        echo "Email to $recipient: $message\n";
    }
}

class SMSChannel implements NotificationChannel {
    public function send(string $recipient, string $message): void {
        // Send SMS
        echo "SMS to $recipient: $message\n";
    }
}

// Open/Closed: Add channels without modifying NotificationService
class PushChannel implements NotificationChannel {
    public function send(string $recipient, string $message): void {
        echo "Push to $recipient: $message\n";
    }
}

// Dependency Inversion: Depends on abstraction
class NotificationService {
    /** @param NotificationChannel[] $channels */
    public function __construct(
        private array $channels
    ) {}
    
    public function notify(string $recipient, string $message): void {
        foreach ($this->channels as $channel) {
            $channel->send($recipient, $message);
        }
    }
}

// Usage
$notifier = new NotificationService([
    new EmailChannel(),
    new SMSChannel(),
    new PushChannel(),
]);

$notifier->notify('user@example.com', 'Hello!');
?>
```


---

> 📘 *This lesson is part of the [Object-Oriented PHP Mastery](https://stanza.dev/courses/php-oop-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*