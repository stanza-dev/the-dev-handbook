---
source_course: "php-oop-mastery"
source_lesson: "php-oop-mastery-interfaces-deep"
---

# Interfaces Deep Dive

Interfaces define contracts that classes must fulfill. They specify WHAT a class must do, not HOW.

## Basic Interface

```php
<?php
interface Printable {
    public function print(): string;
}

class Invoice implements Printable {
    public function __construct(
        private float $amount
    ) {}
    
    public function print(): string {
        return "Invoice: \${$this->amount}";
    }
}

class Report implements Printable {
    public function __construct(
        private string $title
    ) {}
    
    public function print(): string {
        return "Report: {$this->title}";
    }
}
```

## Multiple Interface Implementation

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

// Implement multiple interfaces
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

## Interface Inheritance

```php
<?php
interface Readable {
    public function read(): string;
}

interface Writable {
    public function write(string $data): void;
}

// Interface extending multiple interfaces
interface ReadWritable extends Readable, Writable {
    public function isOpen(): bool;
}

class FileStream implements ReadWritable {
    public function read(): string { /* ... */ }
    public function write(string $data): void { /* ... */ }
    public function isOpen(): bool { /* ... */ }
}
```

## Interface Constants

```php
<?php
interface HttpStatus {
    public const OK = 200;
    public const NOT_FOUND = 404;
    public const SERVER_ERROR = 500;
}

class Response implements HttpStatus {
    public function __construct(
        private int $status = self::OK
    ) {}
}
```

## Type Hinting with Interfaces

```php
<?php
interface Logger {
    public function log(string $message): void;
}

class Application {
    public function __construct(
        private Logger $logger  // Accept any Logger implementation
    ) {}
    
    public function run(): void {
        $this->logger->log('Application started');
    }
}

// Dependency injection with interfaces
$app = new Application(new FileLogger());
$app = new Application(new ConsoleLogger());
$app = new Application(new NullLogger());  // For testing
```

## Resources

- [Object Interfaces](https://www.php.net/manual/en/language.oop5.interfaces.php) — PHP interfaces documentation

---

> 📘 *This lesson is part of the [Object-Oriented PHP Mastery](https://stanza.dev/courses/php-oop-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*