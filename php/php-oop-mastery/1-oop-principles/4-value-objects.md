---
source_course: "php-oop-mastery"
source_lesson: "php-oop-mastery-value-objects"
---

# Value Objects Pattern

Value Objects are small, immutable objects that represent a descriptive aspect of the domain. They are defined by their attributes, not an identity.

## Entity vs Value Object

| Entity | Value Object |
|--------|-------------|
| Has unique identity | Identified by value |
| Mutable | Immutable |
| Same ID = same thing | Same values = equal |
| Example: User, Order | Example: Money, Email |

## Implementing Value Objects

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

## Money Value Object

```php
<?php
readonly class Money
{
    public function __construct(
        public int $amount,      // In smallest unit (cents)
        public string $currency
    ) {
        if ($amount < 0) {
            throw new InvalidArgumentException('Amount cannot be negative');
        }
        
        if (!in_array($currency, ['USD', 'EUR', 'GBP'], true)) {
            throw new InvalidArgumentException('Unsupported currency');
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
    
    public function format(): string
    {
        $symbols = ['USD' => '$', 'EUR' => '€', 'GBP' => '£'];
        return $symbols[$this->currency] . number_format($this->amount / 100, 2);
    }
    
    private function assertSameCurrency(Money $other): void
    {
        if ($this->currency !== $other->currency) {
            throw new CurrencyMismatchException();
        }
    }
}
```

## Date Range Value Object

```php
<?php
readonly class DateRange
{
    public function __construct(
        public DateTimeImmutable $start,
        public DateTimeImmutable $end
    ) {
        if ($end < $start) {
            throw new InvalidArgumentException('End date must be after start date');
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
    
    public function getDays(): int
    {
        return $this->start->diff($this->end)->days;
    }
    
    public function extend(DateInterval $interval): self
    {
        return new self($this->start, $this->end->add($interval));
    }
}
```

## When to Use Value Objects

- **Email addresses**: Validation, formatting
- **Money/Currency**: Arithmetic with safety
- **Coordinates**: Lat/long with calculations
- **Addresses**: Formatting, validation
- **Measurements**: Units, conversions

## Resources

- [Value Objects](https://martinfowler.com/bliki/ValueObject.html) — Martin Fowler on Value Objects

---

> 📘 *This lesson is part of the [Object-Oriented PHP Mastery](https://stanza.dev/courses/php-oop-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*