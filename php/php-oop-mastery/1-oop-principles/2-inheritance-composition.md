---
source_course: "php-oop-mastery"
source_lesson: "php-oop-mastery-inheritance-composition"
---

# Inheritance vs Composition

## Introduction
Inheritance and composition are two fundamental approaches to code reuse and building relationships between classes. Understanding when to use each is critical for designing maintainable PHP applications.

## Key Concepts
- **Inheritance (IS-A)**: A child class extends a parent class and inherits its properties and methods.
- **Composition (HAS-A)**: A class contains instances of other classes as properties and delegates behavior to them.
- **Fragile Base Class Problem**: Changes to a parent class can unexpectedly break child classes.
- **Favor Composition Over Inheritance**: A widely accepted principle promoting flexibility.

## Real World Context
Consider a CMS. You might create `Content > Article > BlogPost > FeaturedBlogPost`. When you need a `FeaturedVideo`, the hierarchy is too rigid. Composition lets you mix capabilities like `Featurable`, `Publishable`, and `Commentable` without deep chains.

## Deep Dive

### Inheritance (IS-A Relationship)

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
```

This works well for shallow, stable hierarchies where subtype polymorphism is truly needed.

### Problems with Deep Inheritance

```php
<?php
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
```

Any change to `Vehicle` or `Car` can cascade down and break `ElectricCar`.

### Composition (HAS-A Relationship)

```php
<?php
interface EngineInterface {
    public function start(): void;
    public function stop(): void;
}

class GasEngine implements EngineInterface {
    public function start(): void { echo "Gas engine started\n"; }
    public function stop(): void { echo "Gas engine stopped\n"; }
}

class ElectricEngine implements EngineInterface {
    public function start(): void { echo "Electric motor humming\n"; }
    public function stop(): void { echo "Electric motor stopped\n"; }
}

class Car {
    public function __construct(
        private EngineInterface $engine
    ) {}

    public function start(): void { $this->engine->start(); }
    public function stop(): void { $this->engine->stop(); }
}

$gasCar = new Car(new GasEngine());
$electricCar = new Car(new ElectricEngine());
```

With composition, you can swap the engine without changing the `Car` class.

### When to Use Each

| Use Inheritance When | Use Composition When |
|---------------------|---------------------|
| True IS-A relationship | HAS-A or USES-A relationship |
| Sharing implementation | Sharing behavior/contract |
| Subtype polymorphism needed | Flexibility needed |
| Shallow hierarchy (1-2 levels) | Deep hierarchies |
| Framework extension points | Business logic |

## Common Pitfalls
1. **Deep inheritance hierarchies** — More than 2-3 levels deep almost always signals a design problem. Refactor to composition.
2. **Using inheritance for code reuse alone** — If there is no true IS-A relationship, use composition or traits instead.

## Best Practices
1. **Start with composition** — Default to injecting dependencies and only use inheritance when subtype polymorphism is genuinely needed.
2. **Keep hierarchies shallow** — Limit inheritance to 1-2 levels and consider making base classes abstract.

## Summary
- Inheritance models IS-A relationships but creates tight coupling in deep hierarchies.
- Composition models HAS-A relationships with greater flexibility and testability.
- Favor composition over inheritance for business logic.
- Use interfaces to define contracts and enable dependency swapping.

## Code Examples

**Composition-based notification system**

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

class SmsChannel implements NotificationChannel {
    public function send(string $recipient, string $message): void {
        echo "SMS to $recipient: $message\n";
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

$notifier = new NotificationService([new EmailChannel(), new SmsChannel()]);
$notifier->notify('user@example.com', 'Hello!');
?>
```


## Resources

- [Object Inheritance](https://www.php.net/manual/en/language.oop5.inheritance.php) — PHP inheritance documentation

---

> 📘 *This lesson is part of the [Object-Oriented PHP Mastery](https://stanza.dev/courses/php-oop-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*