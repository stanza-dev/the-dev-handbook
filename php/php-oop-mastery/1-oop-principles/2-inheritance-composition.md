---
source_course: "php-oop-mastery"
source_lesson: "php-oop-mastery-inheritance-composition"
---

# Inheritance vs Composition

Two fundamental ways to reuse code and build relationships between classes.

## Inheritance (IS-A Relationship)

```php
<?php
class Animal {
    protected string $name;
    
    public function __construct(string $name) {
        $this->name = $name;
    }
    
    public function speak(): string {
        return 'Some sound';
    }
}

class Dog extends Animal {
    public function speak(): string {
        return 'Woof!';
    }
    
    public function fetch(): void {
        echo "{$this->name} is fetching!";
    }
}

$dog = new Dog('Rex');
echo $dog->speak();  // Woof!
$dog->fetch();       // Rex is fetching!
```

## Problems with Deep Inheritance

```php
<?php
// Fragile base class problem
class Vehicle {
    public function start(): void { /* ... */ }
    public function stop(): void { /* ... */ }
}

class Car extends Vehicle {
    public function honk(): void { /* ... */ }
}

class ElectricCar extends Car {
    // Changes to Vehicle or Car can break this!
}

// Tight coupling, hard to test, inflexible
```

## Composition (HAS-A Relationship)

```php
<?php
// Instead of inheritance, use composition
class Engine {
    public function start(): void {
        echo "Engine started\n";
    }
    
    public function stop(): void {
        echo "Engine stopped\n";
    }
}

class Car {
    public function __construct(
        private Engine $engine
    ) {}
    
    public function start(): void {
        $this->engine->start();
    }
    
    public function stop(): void {
        $this->engine->stop();
    }
}

// Easy to swap engines!
class ElectricEngine {
    public function start(): void {
        echo "Electric motor humming\n";
    }
    
    public function stop(): void {
        echo "Electric motor stopped\n";
    }
}
```

## Favor Composition Over Inheritance

```php
<?php
// Inheritance approach (rigid)
class Logger {
    public function log(string $message): void { /* ... */ }
}

class FileLogger extends Logger {
    // Tied to Logger implementation
}

// Composition approach (flexible)
interface LoggerInterface {
    public function log(string $message): void;
}

class Application {
    public function __construct(
        private LoggerInterface $logger  // Inject dependency
    ) {}
    
    public function doSomething(): void {
        $this->logger->log('Doing something');
    }
}

// Can swap any LoggerInterface implementation
$app = new Application(new FileLogger());
$app = new Application(new DatabaseLogger());
$app = new Application(new NullLogger());  // For testing
```

## When to Use Each

| Use Inheritance When | Use Composition When |
|---------------------|---------------------|
| True IS-A relationship | HAS-A or USES-A relationship |
| Sharing implementation | Sharing behavior/contract |
| Subtype polymorphism needed | Flexibility needed |
| Shallow hierarchy (1-2 levels) | Deep hierarchies |
| Framework extension points | Business logic |

## Resources

- [Object Inheritance](https://www.php.net/manual/en/language.oop5.inheritance.php) — PHP inheritance documentation

---

> 📘 *This lesson is part of the [Object-Oriented PHP Mastery](https://stanza.dev/courses/php-oop-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*