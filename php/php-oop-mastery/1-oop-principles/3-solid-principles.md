---
source_course: "php-oop-mastery"
source_lesson: "php-oop-mastery-solid-principles"
---

# SOLID Principles

## Introduction
SOLID is an acronym for five design principles that help developers write maintainable, flexible, and scalable object-oriented code. These principles were popularized by Robert C. Martin and form the foundation of good OOP design.

## Key Concepts
- **SRP**: A class should have only one reason to change.
- **OCP**: Open for extension, closed for modification.
- **LSP**: Subtypes must be substitutable for their base types.
- **ISP**: Clients should not depend on interfaces they do not use.
- **DIP**: Depend on abstractions, not concretions.

## Real World Context
In a large e-commerce application, violating SOLID leads to classes that are hundreds of lines long, changes that break unrelated features, and impossible-to-write tests. Following SOLID keeps each class focused, changes localized, and testing straightforward.

## Deep Dive

### S - Single Responsibility Principle

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

Each class has one job and one reason to change.

### O - Open/Closed Principle

```php
<?php
// BAD: Must modify to add new types
class PaymentProcessor {
    public function process(string $type, float $amount): void {
        if ($type === 'credit') { /* ... */ }
        elseif ($type === 'paypal') { /* ... */ }
    }
}

// GOOD: Extend without modifying
interface PaymentGateway {
    public function process(float $amount): void;
}

class CreditCardGateway implements PaymentGateway {
    public function process(float $amount): void { /* ... */ }
}

class CryptoGateway implements PaymentGateway {
    public function process(float $amount): void { /* ... */ }
}
```

New payment types are added by creating new classes, not modifying existing ones.

### L - Liskov Substitution Principle

```php
<?php
// BAD: Square violates LSP
class Rectangle {
    protected int $width;
    protected int $height;

    public function setWidth(int $w): void { $this->width = $w; }
    public function setHeight(int $h): void { $this->height = $h; }
    public function getArea(): int { return $this->width * $this->height; }
}

class Square extends Rectangle {
    public function setWidth(int $w): void {
        $this->width = $w;
        $this->height = $w;  // Breaks expectations!
    }
}
```

Code expecting `Rectangle` behavior produces wrong results with `Square`.

### I - Interface Segregation Principle

```php
<?php
// BAD: Fat interface
interface Worker {
    public function work(): void;
    public function eat(): void;
    public function sleep(): void;
}

// GOOD: Segregated interfaces
interface Workable { public function work(): void; }
interface Eatable { public function eat(): void; }

class Human implements Workable, Eatable { /* ... */ }
class Robot implements Workable { /* ... */ }  // Only what it needs
```

### D - Dependency Inversion Principle

```php
<?php
// BAD: Depends on concrete class
class OrderService {
    private MySQLDatabase $db;
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
        private DatabaseInterface $db
    ) {}
}
```

Depending on abstractions makes code flexible and testable.

## Common Pitfalls
1. **Over-engineering SRP** — Do not create a class for every single method. Group related responsibilities that change for the same reason.
2. **Violating LSP with exceptions** — Throwing `NotImplementedException` in a subclass signals an LSP violation.

## Best Practices
1. **Apply SOLID incrementally** — Refactor toward SOLID as complexity grows rather than over-designing upfront.
2. **Use interfaces for DIP** — Always depend on interfaces or abstract classes for external dependencies.

## Summary
- SRP keeps classes focused with one reason to change.
- OCP enables extension without modification through polymorphism.
- LSP ensures subtypes can replace base types without breaking behavior.
- ISP keeps interfaces small and focused.
- DIP promotes depending on abstractions for flexibility and testability.

## Code Examples

**SOLID-compliant notification system**

```php
<?php
declare(strict_types=1);

interface NotificationChannel {
    public function send(string $recipient, string $message): void;
}

class EmailChannel implements NotificationChannel {
    public function send(string $recipient, string $message): void {
        echo "Email to $recipient: $message\n";
    }
}

class SMSChannel implements NotificationChannel {
    public function send(string $recipient, string $message): void {
        echo "SMS to $recipient: $message\n";
    }
}

class PushChannel implements NotificationChannel {
    public function send(string $recipient, string $message): void {
        echo "Push to $recipient: $message\n";
    }
}

class NotificationService {
    /** @param NotificationChannel[] $channels */
    public function __construct(private array $channels) {}

    public function notify(string $recipient, string $message): void {
        foreach ($this->channels as $channel) {
            $channel->send($recipient, $message);
        }
    }
}

$notifier = new NotificationService([
    new EmailChannel(),
    new SMSChannel(),
    new PushChannel(),
]);
$notifier->notify('user@example.com', 'Hello!');
?>
```


## Resources

- [SOLID Principles](https://en.wikipedia.org/wiki/SOLID) — SOLID principles overview

---

> 📘 *This lesson is part of the [Object-Oriented PHP Mastery](https://stanza.dev/courses/php-oop-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*