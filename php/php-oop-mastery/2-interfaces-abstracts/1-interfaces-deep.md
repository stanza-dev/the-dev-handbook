---
source_course: "php-oop-mastery"
source_lesson: "php-oop-mastery-interfaces-deep"
---

# Interfaces Deep Dive

## Introduction
Interfaces define contracts that classes must fulfill. They specify WHAT a class must do, not HOW. Interfaces are fundamental to writing loosely coupled, testable PHP code.

## Key Concepts
- **Contract**: An interface guarantees a class provides certain methods.
- **Multiple Implementation**: A class can implement multiple interfaces, unlike single class inheritance.
- **Interface Inheritance**: Interfaces can extend other interfaces to build complex contracts.
- **Type Hinting**: Interfaces enable polymorphism through type hints.

## Real World Context
In a payment processing system, you need to support credit cards, PayPal, and cryptocurrency. A `PaymentGateway` interface defines a `charge()` method. Each provider implements it differently, but your checkout code only depends on the interface.

## Deep Dive

### Basic Interface

```php
<?php
interface Printable {
    public function print(): string;
}

class Invoice implements Printable {
    public function __construct(private float $amount) {}

    public function print(): string {
        return "Invoice: \${$this->amount}";
    }
}

class Report implements Printable {
    public function __construct(private string $title) {}

    public function print(): string {
        return "Report: {$this->title}";
    }
}
```

Both classes guarantee they have a `print()` method, but each implements it differently.

### Multiple Interface Implementation

```php
<?php
interface Serializable {
    public function serialize(): string;
}

interface Cacheable {
    public function getCacheKey(): string;
    public function getTtl(): int;
}

interface Loggable {
    public function getLogContext(): array;
}

class UserSession implements Serializable, Cacheable, Loggable {
    public function __construct(
        private string $userId,
        private array $data
    ) {}

    public function serialize(): string {
        return json_encode($this->data);
    }

    public function getCacheKey(): string {
        return "session:{$this->userId}";
    }

    public function getTtl(): int {
        return 3600;
    }

    public function getLogContext(): array {
        return ['user_id' => $this->userId];
    }
}
```

This is more flexible than multiple inheritance, which PHP does not support.

### Interface Inheritance

```php
<?php
interface Readable {
    public function read(): string;
}

interface Writable {
    public function write(string $data): void;
}

interface ReadWritable extends Readable, Writable {
    public function isOpen(): bool;
}

class FileStream implements ReadWritable {
    public function read(): string { return ''; }
    public function write(string $data): void { /* ... */ }
    public function isOpen(): bool { return true; }
}
```

A class implementing `ReadWritable` must provide all methods from both parent interfaces.

### Type Hinting with Interfaces

```php
<?php
interface Logger {
    public function log(string $message): void;
}

class Application {
    public function __construct(private Logger $logger) {}

    public function run(): void {
        $this->logger->log('Application started');
    }
}

$app = new Application(new FileLogger());
$app = new Application(new NullLogger());  // For testing
```

The class works with any `Logger` implementation without knowing the specific class.

## Common Pitfalls
1. **Fat interfaces** — An interface with too many methods forces implementors to provide methods they do not need.
2. **Single-implementation interfaces** — Creating an interface with only one implementation adds complexity without benefit.

## Best Practices
1. **Keep interfaces small** — Follow the Interface Segregation Principle with multiple small interfaces.
2. **Name by capability** — Use names like `Printable`, `Cacheable`, `Serializable` that describe what the implementor can do.

## Summary
- Interfaces define contracts that classes must fulfill.
- A class can implement multiple interfaces for flexible polymorphism.
- Interfaces can extend other interfaces to compose contracts.
- Type-hint against interfaces for loose coupling and testability.

## Code Examples

**Intersection types with multiple interfaces**

```php
<?php
declare(strict_types=1);

interface Renderable {
    public function render(): string;
}

interface HasTitle {
    public function getTitle(): string;
}

class HtmlPage implements Renderable, HasTitle {
    public function __construct(
        private string $title,
        private string $body
    ) {}

    public function getTitle(): string { return $this->title; }

    public function render(): string {
        return "<html><head><title>{$this->title}</title></head><body>{$this->body}</body></html>";
    }
}

function displayPage(Renderable&HasTitle $page): void {
    echo "Title: " . $page->getTitle() . "\n";
    echo $page->render();
}

displayPage(new HtmlPage('Welcome', '<h1>Hello</h1>'));
?>
```


## Resources

- [Object Interfaces](https://www.php.net/manual/en/language.oop5.interfaces.php) — PHP interfaces documentation

---

> 📘 *This lesson is part of the [Object-Oriented PHP Mastery](https://stanza.dev/courses/php-oop-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*