---
source_course: "php-oop-mastery"
source_lesson: "php-oop-mastery-value-objects"
---

# Value Objects Pattern

## Introduction
Value Objects are small, immutable objects that represent a descriptive aspect of the domain. Unlike entities, they are defined by their attribute values rather than a unique identity. They are one of the most powerful patterns for writing bug-free domain code.

## Key Concepts
- **Identity vs Value**: Entities have identity (User with ID 42); Value Objects have only values (Money of $5.00).
- **Immutability**: Value Objects never change after creation. Operations return new instances.
- **Self-Validation**: Value Objects validate in the constructor, making invalid states unrepresentable.
- **Equality by Value**: Two Value Objects with the same attributes are considered equal.

## Real World Context
In a payment system, representing money as a bare `float` risks currency mismatches, rounding errors, and negative amounts. A `Money` Value Object encapsulates the amount and currency, validates on construction, and provides safe arithmetic.

## Deep Dive

### Entity vs Value Object

| Entity | Value Object |
|--------|-------------|
| Has unique identity | Identified by value |
| Mutable | Immutable |
| Same ID = same thing | Same values = equal |
| Example: User, Order | Example: Money, Email |

### Implementing Value Objects

PHP's `readonly` classes are perfect for Value Objects.

```php
<?php
readonly class Email
{
    public function __construct(
        public string $address
    ) {
        if (!filter_var($address, FILTER_VALIDATE_EMAIL)) {
            throw new InvalidArgumentException('Invalid email address');
        }
    }

    public function getDomain(): string
    {
        return substr($this->address, strpos($this->address, '@') + 1);
    }

    public function equals(Email $other): bool
    {
        return strtolower($this->address) === strtolower($other->address);
    }

    public function __toString(): string
    {
        return $this->address;
    }
}
```

The constructor validates input immediately. The `readonly` class ensures immutability.

### Money Value Object

```php
<?php
readonly class Money
{
    public function __construct(
        public int $amount,
        public string $currency
    ) {
        if ($amount < 0) {
            throw new InvalidArgumentException('Amount cannot be negative');
        }
    }

    public function add(Money $other): self
    {
        $this->assertSameCurrency($other);
        return new self($this->amount + $other->amount, $this->currency);
    }

    public function subtract(Money $other): self
    {
        $this->assertSameCurrency($other);
        if ($other->amount > $this->amount) {
            throw new InsufficientFundsException();
        }
        return new self($this->amount - $other->amount, $this->currency);
    }

    public function multiply(float $factor): self
    {
        return new self((int) round($this->amount * $factor), $this->currency);
    }

    public function equals(Money $other): bool
    {
        return $this->amount === $other->amount
            && $this->currency === $other->currency;
    }

    private function assertSameCurrency(Money $other): void
    {
        if ($this->currency !== $other->currency) {
            throw new CurrencyMismatchException();
        }
    }
}
```

Operations like `add()` return new instances instead of modifying the original.

### Date Range Value Object

```php
<?php
readonly class DateRange
{
    public function __construct(
        public DateTimeImmutable $start,
        public DateTimeImmutable $end
    ) {
        if ($end < $start) {
            throw new InvalidArgumentException('End must be after start');
        }
    }

    public function contains(DateTimeImmutable $date): bool
    {
        return $date >= $this->start && $date <= $this->end;
    }

    public function overlaps(DateRange $other): bool
    {
        return $this->start <= $other->end && $this->end >= $other->start;
    }
}
```

Self-validation makes it impossible to create a date range where end is before start.

## Common Pitfalls
1. **Using primitive types for domain concepts** — A bare `string` for email allows invalid values. Wrap it in a Value Object.
2. **Making Value Objects mutable** — If a Value Object can change, shared references lead to subtle bugs. Always use `readonly`.

## Best Practices
1. **Validate in the constructor** — Throw exceptions for invalid data immediately. A Value Object should never exist in an invalid state.
2. **Implement equals() explicitly** — PHP does not compare objects by value by default. Provide an explicit `equals()` method.

## Summary
- Value Objects are identified by their attribute values, not identity.
- They must be immutable; operations return new instances.
- Self-validation in the constructor prevents invalid states.
- Use them for domain concepts like money, email, dates, and coordinates.

## Code Examples

**Coordinate Value Object with distance calculation**

```php
<?php
declare(strict_types=1);

readonly class Coordinate
{
    public function __construct(
        public float $latitude,
        public float $longitude
    ) {
        if ($latitude < -90 || $latitude > 90) {
            throw new InvalidArgumentException('Invalid latitude');
        }
        if ($longitude < -180 || $longitude > 180) {
            throw new InvalidArgumentException('Invalid longitude');
        }
    }

    public function distanceTo(Coordinate $other): float
    {
        $r = 6371;
        $dLat = deg2rad($other->latitude - $this->latitude);
        $dLon = deg2rad($other->longitude - $this->longitude);
        $a = sin($dLat/2)**2 + cos(deg2rad($this->latitude))
            * cos(deg2rad($other->latitude)) * sin($dLon/2)**2;
        return $r * 2 * atan2(sqrt($a), sqrt(1-$a));
    }

    public function equals(Coordinate $other): bool
    {
        return $this->latitude === $other->latitude
            && $this->longitude === $other->longitude;
    }
}

$paris = new Coordinate(48.8566, 2.3522);
$london = new Coordinate(51.5074, -0.1278);
echo $paris->distanceTo($london) . ' km';
?>
```


## Resources

- [Value Objects](https://martinfowler.com/bliki/ValueObject.html) — Martin Fowler on Value Objects

---

> 📘 *This lesson is part of the [Object-Oriented PHP Mastery](https://stanza.dev/courses/php-oop-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*