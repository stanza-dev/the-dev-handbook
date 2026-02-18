---
source_course: "php-modern-features"
source_lesson: "php-modern-features-enum-patterns"
---

# Advanced Enum Patterns

Enums support advanced patterns including state machines, interface implementations, and trait usage.

## State Machine with Enums

```php
<?php
enum OrderStatus: string
{
    case Pending = 'pending';
    case Confirmed = 'confirmed';
    case Shipped = 'shipped';
    case Delivered = 'delivered';
    case Cancelled = 'cancelled';
    
    public function canTransitionTo(self $next): bool
    {
        return match ($this) {
            self::Pending => in_array($next, [self::Confirmed, self::Cancelled]),
            self::Confirmed => in_array($next, [self::Shipped, self::Cancelled]),
            self::Shipped => $next === self::Delivered,
            self::Delivered, self::Cancelled => false,
        };
    }
    
    public function transitionTo(self $next): self
    {
        if (!$this->canTransitionTo($next)) {
            throw new InvalidArgumentException(
                "Cannot transition from {$this->value} to {$next->value}"
            );
        }
        return $next;
    }
    
    public function isFinal(): bool
    {
        return in_array($this, [self::Delivered, self::Cancelled]);
    }
}

// Usage
$status = OrderStatus::Pending;
$status = $status->transitionTo(OrderStatus::Confirmed); // OK
$status = $status->transitionTo(OrderStatus::Shipped);   // OK
```

## Enums Implementing Interfaces

```php
<?php
interface Describable
{
    public function describe(): string;
    public function getIcon(): string;
}

enum Permission: string implements Describable
{
    case Read = 'read';
    case Write = 'write';
    case Delete = 'delete';
    case Admin = 'admin';
    
    public function describe(): string
    {
        return match ($this) {
            self::Read => 'View and read content',
            self::Write => 'Create and edit content',
            self::Delete => 'Remove content permanently',
            self::Admin => 'Full administrative access',
        };
    }
    
    public function getIcon(): string
    {
        return match ($this) {
            self::Read => 'eye',
            self::Write => 'pencil',
            self::Delete => 'trash',
            self::Admin => 'crown',
        };
    }
}
```

## Enum with Traits

```php
<?php
trait EnumToArray
{
    public static function names(): array
    {
        return array_column(self::cases(), 'name');
    }
    
    public static function values(): array
    {
        return array_column(self::cases(), 'value');
    }
    
    public static function toArray(): array
    {
        return array_combine(self::names(), self::values());
    }
}

enum Color: string
{
    use EnumToArray;
    
    case Red = '#FF0000';
    case Green = '#00FF00';
    case Blue = '#0000FF';
}

Color::toArray();
// ['Red' => '#FF0000', 'Green' => '#00FF00', 'Blue' => '#0000FF']
```

## Enum Validation and Forms

```php
<?php
enum Country: string
{
    case US = 'us';
    case UK = 'uk';
    case DE = 'de';
    
    public function label(): string
    {
        return match ($this) {
            self::US => 'United States',
            self::UK => 'United Kingdom',
            self::DE => 'Germany',
        };
    }
    
    public static function forSelect(): array
    {
        return array_map(
            fn($case) => ['value' => $case->value, 'label' => $case->label()],
            self::cases()
        );
    }
    
    public static function isValid(string $value): bool
    {
        return self::tryFrom($value) !== null;
    }
}
```

## Resources

- [Enumerations](https://www.php.net/manual/en/language.enumerations.php) — PHP enumerations documentation

---

> 📘 *This lesson is part of the [Modern PHP 8.x: Latest Language Features](https://stanza.dev/courses/php-modern-features) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*