---
source_course: "php-oop-mastery"
source_lesson: "php-oop-mastery-factory-pattern"
---

# Factory Pattern

## Introduction
The Factory pattern creates objects without specifying their exact class. It centralizes and encapsulates object creation logic, making code more flexible and testable.

## Key Concepts
- **Simple Factory**: A single class with a static method that creates objects based on a parameter.
- **Factory Method**: An abstract class defines a creation method that subclasses override.
- **Abstract Factory**: A family of related factories that produce compatible objects.
- **Encapsulated Creation**: Client code does not need to know the concrete class being created.

## Real World Context
In a logging system, you need different logger implementations for development (console), staging (file), and production (database). A `LoggerFactory` creates the right logger based on the environment, keeping your application code clean and environment-agnostic.

## Deep Dive

### Simple Factory

```php
<?php
interface Logger {
    public function log(string $message): void;
}

class FileLogger implements Logger {
    public function log(string $message): void {
        file_put_contents('app.log', $message . "\n", FILE_APPEND);
    }
}

class DatabaseLogger implements Logger {
    public function log(string $message): void { /* Save to DB */ }
}

class ConsoleLogger implements Logger {
    public function log(string $message): void {
        echo "[LOG] $message\n";
    }
}

class LoggerFactory {
    public static function create(string $type): Logger {
        return match($type) {
            'file' => new FileLogger(),
            'database' => new DatabaseLogger(),
            'console' => new ConsoleLogger(),
            default => throw new InvalidArgumentException("Unknown: $type"),
        };
    }
}

$logger = LoggerFactory::create('file');
$logger->log('Application started');
```

The factory centralizes the creation logic. Adding a new logger type requires changing only the factory.

### Factory Method Pattern

```php
<?php
abstract class Document {
    abstract public function createPages(): array;

    public function render(): string {
        $output = '';
        foreach ($this->createPages() as $page) {
            $output .= $page->render();
        }
        return $output;
    }
}

class Resume extends Document {
    public function createPages(): array {
        return [new SkillsPage(), new ExperiencePage()];
    }
}

class Report extends Document {
    public function createPages(): array {
        return [new IntroductionPage(), new ResultsPage()];
    }
}
```

Each subclass decides which concrete pages to create while the base class defines the rendering algorithm.

### Abstract Factory

```php
<?php
interface Button { public function render(): string; }
interface Checkbox { public function render(): string; }

interface UIFactory {
    public function createButton(): Button;
    public function createCheckbox(): Checkbox;
}

class DarkButton implements Button {
    public function render(): string {
        return '<button class="dark">Click</button>';
    }
}

class DarkCheckbox implements Checkbox {
    public function render(): string {
        return '<input type="checkbox" class="dark">';
    }
}

class DarkUIFactory implements UIFactory {
    public function createButton(): Button { return new DarkButton(); }
    public function createCheckbox(): Checkbox { return new DarkCheckbox(); }
}

function renderForm(UIFactory $factory): string {
    return $factory->createCheckbox()->render() . $factory->createButton()->render();
}
```

Abstract Factory ensures that related objects (dark button + dark checkbox) are always used together.

## Common Pitfalls
1. **Overusing factories** — Not every object needs a factory. Use them when creation logic is complex or varies.
2. **God factory** — A factory that creates everything becomes a maintenance burden. Keep factories focused.

## Best Practices
1. **Use match expressions** — PHP 8's match is cleaner than switch for simple factory methods.
2. **Return interfaces** — Factories should return interface types, not concrete classes.

## Summary
- Simple Factory centralizes object creation in a single class.
- Factory Method lets subclasses decide which concrete class to create.
- Abstract Factory produces families of related objects.
- Factories hide creation complexity and promote loose coupling.

## Code Examples

**Cache driver factory pattern**

```php
<?php
declare(strict_types=1);

interface CacheDriver {
    public function get(string $key): mixed;
    public function set(string $key, mixed $value, int $ttl = 3600): void;
}

class MemcachedDriver implements CacheDriver {
    public function get(string $key): mixed { return null; }
    public function set(string $key, mixed $value, int $ttl = 3600): void {}
}

class RedisDriver implements CacheDriver {
    public function get(string $key): mixed { return null; }
    public function set(string $key, mixed $value, int $ttl = 3600): void {}
}

class ArrayDriver implements CacheDriver {
    private array $store = [];
    public function get(string $key): mixed { return $this->store[$key] ?? null; }
    public function set(string $key, mixed $value, int $ttl = 3600): void {
        $this->store[$key] = $value;
    }
}

class CacheFactory {
    public static function create(string $driver): CacheDriver {
        return match($driver) {
            'memcached' => new MemcachedDriver(),
            'redis' => new RedisDriver(),
            'array' => new ArrayDriver(),
            default => throw new InvalidArgumentException("Unknown driver: $driver"),
        };
    }
}

$cache = CacheFactory::create('redis');
$cache->set('user:1', ['name' => 'John']);
?>
```


## Resources

- [Factory Pattern](https://refactoring.guru/design-patterns/factory-method/php/example) — Factory pattern examples

---

> 📘 *This lesson is part of the [Object-Oriented PHP Mastery](https://stanza.dev/courses/php-oop-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*