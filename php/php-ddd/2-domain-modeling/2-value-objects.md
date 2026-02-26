---
source_course: "php-ddd"
source_lesson: "php-ddd-value-objects"
---

# Value Objects

## Introduction

A **Value Object** is a domain object defined entirely by its attributes. Two value objects with the same attributes are considered equal and interchangeable.

## Key Concepts

- **Equality by value**
- **Immutable**
- **Replaceable**
- **Self-validating**
- **Side-effect free**
- **Value Object**

## Real World Context

In production PHP applications, value objects helps teams build maintainable software by providing clear patterns for organizing complex business logic.

## Deep Dive

### Value Objects vs Entities
```php
<?php
// VALUE OBJECT: Money - defined by amount and currency
$money1 = new Money(100, 'USD');
$money2 = new Money(100, 'USD');
$money1->equals($money2);  // true - same value

// ENTITY: Customer - defined by identity
$customer1 = new Customer(new CustomerId('1'), 'John');
$customer2 = new Customer(new CustomerId('2'), 'John');
$customer1->equals($customer2);  // false - different identity
```

### Implementing Value Objects
```php
<?php
declare(strict_types=1);

namespace Domain\Shared;

final readonly class Money {
    public function __construct(
        private int $cents,
        private Currency $currency
    ) {
        if ($cents < 0) {
            throw new InvalidArgumentException(
                'Money cannot be negative'
            );
        }
    }
    
    public static function USD(int $cents): self {
        return new self($cents, Currency::USD);
    }
    
    public static function EUR(int $cents): self {
        return new self($cents, Currency::EUR);
    }
    
    public static function zero(Currency $currency = Currency::USD): self {
        return new self(0, $currency);
    }
    
    public function add(Money $other): self {
        $this->assertSameCurrency($other);
        return new self(
            $this->cents + $other->cents,
            $this->currency
        );
    }
    
    public function subtract(Money $other): self {
        $this->assertSameCurrency($other);
        $result = $this->cents - $other->cents;
        
        if ($result < 0) {
            throw new InvalidArgumentException(
                'Subtraction would result in negative money'
            );
        }
        
        return new self($result, $this->currency);
    }
    
    public function multiply(float $factor): self {
        return new self(
            (int) round($this->cents * $factor),
            $this->currency
        );
    }
    
    public function isGreaterThan(Money $other): bool {
        $this->assertSameCurrency($other);
        return $this->cents > $other->cents;
    }
    
    public function isGreaterThanOrEqual(Money $other): bool {
        $this->assertSameCurrency($other);
        return $this->cents >= $other->cents;
    }
    
    public function cents(): int {
        return $this->cents;
    }
    
    public function currency(): Currency {
        return $this->currency;
    }
    
    public function format(): string {
        $dollars = $this->cents / 100;
        return $this->currency->symbol() . number_format($dollars, 2);
    }
    
    public function equals(Money $other): bool {
        return $this->cents === $other->cents
            && $this->currency === $other->currency;
    }
    
    private function assertSameCurrency(Money $other): void {
        if ($this->currency !== $other->currency) {
            throw new CurrencyMismatchException(
                $this->currency,
                $other->currency
            );
        }
    }
}
```

### Common Value Objects
### Email Address

```php
<?php
final readonly class EmailAddress {
    private string $value;
    
    public function __construct(string $email) {
        $email = strtolower(trim($email));
        
        if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
            throw new InvalidEmailAddress($email);
        }
        
        $this->value = $email;
    }
    
    public function toString(): string {
        return $this->value;
    }
    
    public function domain(): string {
        return substr($this->value, strpos($this->value, '@') + 1);
    }
    
    public function equals(EmailAddress $other): bool {
        return $this->value === $other->value;
    }
}
```

### Address

```php
<?php
final readonly class Address {
    public function __construct(
        private string $street,
        private string $city,
        private string $state,
        private string $postalCode,
        private Country $country
    ) {
        if (empty($street) || empty($city)) {
            throw new InvalidAddress('Street and city are required');
        }
    }
    
    public function withStreet(string $street): self {
        return new self(
            $street,
            $this->city,
            $this->state,
            $this->postalCode,
            $this->country
        );
    }
    
    public function format(): string {
        return implode("\n", [
            $this->street,
            "{$this->city}, {$this->state} {$this->postalCode}",
            $this->country->name()
        ]);
    }
    
    public function equals(Address $other): bool {
        return $this->street === $other->street
            && $this->city === $other->city
            && $this->state === $other->state
            && $this->postalCode === $other->postalCode
            && $this->country === $other->country;
    }
}
```

### DateRange

```php
<?php
final readonly class DateRange {
    public function __construct(
        private \DateTimeImmutable $start,
        private \DateTimeImmutable $end
    ) {
        if ($end < $start) {
            throw new InvalidDateRange(
                'End date must be after start date'
            );
        }
    }
    
    public static function fromStrings(string $start, string $end): self {
        return new self(
            new \DateTimeImmutable($start),
            new \DateTimeImmutable($end)
        );
    }
    
    public function contains(\DateTimeImmutable $date): bool {
        return $date >= $this->start && $date <= $this->end;
    }
    
    public function overlaps(DateRange $other): bool {
        return $this->start <= $other->end 
            && $this->end >= $other->start;
    }
    
    public function lengthInDays(): int {
        return $this->start->diff($this->end)->days;
    }
}
```

### Value Object Characteristics
| Characteristic | Description |
|----------------|-------------|
| **Immutable** | State never changes after creation |
| **Self-validating** | Invalid state is impossible |
| **Equality by value** | Same attributes = equal objects |
| **Side-effect free** | Operations return new instances |
| **Replaceable** | Can be swapped with equal value objects |

### When to Use Value Objects
```php
<?php
// BAD: Primitive obsession
class Order {
    private string $customerId;    // Just a string?
    private float $total;          // Precision issues?
    private string $currency;      // What currencies valid?
    private string $status;        // What values allowed?
}

// GOOD: Rich value objects
class Order {
    private CustomerId $customerId;    // Validated, type-safe
    private Money $total;              // Handles currency properly
    private OrderStatus $status;       // Enum with defined states
}
```

Use value objects when:
- A concept has multiple related attributes
- Validation rules apply
- Operations make sense (add money, compare dates)
- You want to avoid primitive obsession

## Common Pitfalls

1. **Overcomplicating simple cases** - Not every part of the application needs value objects. Apply it where complexity warrants the investment.
2. **Ignoring the ubiquitous language** - Naming classes and methods without input from domain experts leads to a model that does not reflect the business.
3. **Mixing infrastructure concerns** - Allowing framework dependencies to leak into the domain layer undermines the architectural benefits.

## Best Practices

1. **Start from the domain** - Model the business concepts first, then figure out persistence and infrastructure.
2. **Keep it simple** - Use the simplest pattern that solves the problem. Introduce complexity only when needed.
3. **Collaborate with domain experts** - The model should be shaped by business knowledge, not just technical preferences.

## Summary

- Value Objects is a fundamental concept in Domain-Driven Design
- Proper implementation leads to more maintainable and expressive code
- Always align your implementation with the ubiquitous language
- Apply these patterns where business complexity justifies the investment

## Resources

- [PHP 8.2 Readonly Classes](https://www.php.net/manual/en/language.oop5.basic.php#language.oop5.basic.class.readonly) — PHP readonly classes for immutable value objects

---

> 📘 *This lesson is part of the [Domain-Driven Design with PHP](https://stanza.dev/courses/php-ddd) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*