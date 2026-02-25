---
source_course: "php-modern-features"
source_lesson: "php-modern-features-enum-patterns"
---

# Advanced Enum Patterns

## Introduction
Enums support far more than simple value sets. They can implement interfaces, use traits, model state machines, and generate form options. These patterns make enums a powerful tool for domain modeling.

## Key Concepts
- **State Machine**: An enum where methods control valid transitions between cases.
- **Interface Implementation**: Enums can implement interfaces, enabling polymorphism.
- **Trait Usage**: Enums can use traits to share reusable behavior across multiple enums.

## Real World Context
In production applications, enums model order statuses, permissions, feature flags, and configuration options. The patterns in this lesson are used daily in Symfony and Laravel projects for form generation, authorization, API serialization, and workflow management.

## Deep Dive

### State Machine with Enums

Model valid transitions between states:

```php
<?php
enum OrderStatus: string {
    case Pending = 'pending';
    case Confirmed = 'confirmed';
    case Shipped = 'shipped';
    case Delivered = 'delivered';
    case Cancelled = 'cancelled';
    
    public function canTransitionTo(self $next): bool {
        return match ($this) {
            self::Pending => in_array($next, [self::Confirmed, self::Cancelled]),
            self::Confirmed => in_array($next, [self::Shipped, self::Cancelled]),
            self::Shipped => $next === self::Delivered,
            self::Delivered, self::Cancelled => false,
        };
    }
    
    public function transitionTo(self $next): self {
        if (!$this->canTransitionTo($next)) {
            throw new InvalidArgumentException(
                "Cannot transition from {$this->value} to {$next->value}"
            );
        }
        return $next;
    }
}

$status = OrderStatus::Pending;
$status = $status->transitionTo(OrderStatus::Confirmed); // OK
$status = $status->transitionTo(OrderStatus::Shipped);   // OK
```

The enum itself enforces the business rules about which transitions are valid.

### Enums Implementing Interfaces

Enums can implement interfaces for polymorphism:

```php
<?php
interface Describable {
    public function describe(): string;
    public function getIcon(): string;
}

enum Permission: string implements Describable {
    case Read = 'read';
    case Write = 'write';
    case Delete = 'delete';
    case Admin = 'admin';
    
    public function describe(): string {
        return match ($this) {
            self::Read => 'View and read content',
            self::Write => 'Create and edit content',
            self::Delete => 'Remove content permanently',
            self::Admin => 'Full administrative access',
        };
    }
    
    public function getIcon(): string {
        return match ($this) {
            self::Read => 'eye',
            self::Write => 'pencil',
            self::Delete => 'trash',
            self::Admin => 'crown',
        };
    }
}
```

This lets you type-hint `Describable` and pass any enum that implements it.

### Enum with Traits

Share utility methods across multiple enums:

```php
<?php
trait EnumToArray {
    public static function names(): array {
        return array_column(self::cases(), 'name');
    }
    
    public static function values(): array {
        return array_column(self::cases(), 'value');
    }
    
    public static function toArray(): array {
        return array_combine(self::names(), self::values());
    }
}

enum Color: string {
    use EnumToArray;
    
    case Red = '#FF0000';
    case Green = '#00FF00';
    case Blue = '#0000FF';
}

Color::toArray();
// ['Red' => '#FF0000', 'Green' => '#00FF00', 'Blue' => '#0000FF']
```

The trait provides reusable serialization helpers that work with any backed enum.

### Enum for Form Validation

Generate form options and validate input:

```php
<?php
enum Country: string {
    case US = 'us';
    case UK = 'uk';
    case DE = 'de';
    
    public function label(): string {
        return match ($this) {
            self::US => 'United States',
            self::UK => 'United Kingdom',
            self::DE => 'Germany',
        };
    }
    
    public static function forSelect(): array {
        return array_map(
            fn($case) => ['value' => $case->value, 'label' => $case->label()],
            self::cases()
        );
    }
    
    public static function isValid(string $value): bool {
        return self::tryFrom($value) !== null;
    }
}
```

The `forSelect()` method generates data for HTML dropdowns, while `isValid()` checks input without throwing.

## Common Pitfalls
1. **Adding state to enum cases** — Enum cases are singletons and cannot have instance properties. If you need per-instance data, use a class with an enum property instead.
2. **Expecting enums to auto-serialize to JSON** — Enums do not implement `JsonSerializable` by default. You must explicitly access `->value` or `->name` when encoding to JSON.

## Best Practices
1. **Encapsulate business logic in enum methods** — Transition rules, labels, and permissions belong on the enum itself, not scattered across services.
2. **Use the `EnumToArray` trait pattern** — Create a shared trait for common serialization needs rather than duplicating `names()` and `values()` on every enum.

## Summary
- Enums can implement interfaces and use traits for reusable behavior.
- State machine patterns encapsulate transition rules directly on the enum.
- The `EnumToArray` trait provides reusable serialization helpers.
- Use enums for form validation with `tryFrom()` and `forSelect()` patterns.
- Enum cases are singletons — they cannot hold instance state.

## Code Examples

**Enum implementing an interface with domain-specific methods for alert level handling**

```php
<?php
declare(strict_types=1);

interface HasColor {
    public function color(): string;
}

enum AlertLevel: string implements HasColor {
    case Info = 'info';
    case Warning = 'warning';
    case Error = 'error';
    case Critical = 'critical';
    
    public function color(): string {
        return match($this) {
            self::Info => 'blue',
            self::Warning => 'yellow',
            self::Error => 'red',
            self::Critical => 'purple',
        };
    }
    
    public function shouldNotify(): bool {
        return in_array($this, [self::Error, self::Critical]);
    }
}

$level = AlertLevel::Critical;
echo $level->color();        // purple
echo $level->shouldNotify(); // true
?>
```


## Resources

- [Enumerations](https://www.php.net/manual/en/language.enumerations.php) — PHP enumerations documentation

---

> 📘 *This lesson is part of the [Modern PHP 8.x: Latest Language Features](https://stanza.dev/courses/php-modern-features) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*