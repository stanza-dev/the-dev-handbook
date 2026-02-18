---
source_course: "php-modern-features"
source_lesson: "php-modern-features-readonly-properties"
---

# Readonly Properties and Classes

Readonly properties can only be initialized once, then become immutable. This is perfect for value objects and DTOs.

## Readonly Properties (PHP 8.1+)

```php
<?php
class User {
    public readonly string $id;
    public readonly string $name;
    
    public function __construct(string $id, string $name) {
        $this->id = $id;      // OK - first assignment
        $this->name = $name;  // OK - first assignment
    }
    
    public function setName(string $name): void {
        $this->name = $name;  // Error! Already initialized
    }
}

$user = new User('1', 'John');
echo $user->name;     // John
$user->name = 'Jane'; // Error! Cannot modify readonly property
```

## Rules for Readonly Properties

1. Must have a type declaration
2. Can only be assigned once
3. Can only be assigned from the declaring class
4. Cannot have a default value (unless in constructor promotion)

```php
<?php
class Example {
    // Error! Readonly must have a type
    public readonly $invalid;
    
    // OK with type
    public readonly string $valid;
    
    // Error! Cannot have default value
    public readonly string $withDefault = 'nope';
    
    // OK in constructor promotion
    public function __construct(
        public readonly string $promoted = 'default'
    ) {}
}
```

## Readonly Classes (PHP 8.2+)

All properties are implicitly readonly:

```php
<?php
readonly class Point {
    public function __construct(
        public float $x,
        public float $y
    ) {}
}

// Equivalent to:
class Point {
    public function __construct(
        public readonly float $x,
        public readonly float $y
    ) {}
}

$point = new Point(1.0, 2.0);
$point->x = 3.0;  // Error! Cannot modify
```

## Readonly Class Rules

```php
<?php
readonly class Immutable {
    // All properties must be typed
    public string $name;   // OK
    public $untyped;       // Error!
    
    // No static properties allowed
    public static string $static;  // Error!
}
```

## Cloning Readonly Objects (PHP 8.3+)

```php
<?php
readonly class User {
    public function __construct(
        public string $name,
        public string $email
    ) {}
    
    public function withEmail(string $email): self {
        // PHP 8.3+: Can modify during clone
        return clone $this with {
            $this->email = $email
        };
    }
}

// Alternative: Manual clone method
readonly class User {
    public function __construct(
        public string $name,
        public string $email
    ) {}
    
    public function withEmail(string $email): self {
        return new self($this->name, $email);
    }
}
```

## Code Examples

**Immutable Money value object**

```php
<?php
declare(strict_types=1);

// Value Object pattern with readonly class
readonly class Money {
    public function __construct(
        public int $amount,      // In cents
        public string $currency
    ) {
        if ($amount < 0) {
            throw new InvalidArgumentException('Amount cannot be negative');
        }
    }
    
    public function add(Money $other): self {
        if ($this->currency !== $other->currency) {
            throw new InvalidArgumentException('Currency mismatch');
        }
        return new self($this->amount + $other->amount, $this->currency);
    }
    
    public function multiply(float $factor): self {
        return new self((int) round($this->amount * $factor), $this->currency);
    }
    
    public function format(): string {
        return sprintf('%s %.2f', $this->currency, $this->amount / 100);
    }
}

$price = new Money(1999, 'USD');    // $19.99
$tax = $price->multiply(0.1);       // $1.99 (new object)
$total = $price->add($tax);         // $21.98 (new object)

echo $total->format();  // USD 21.98
?>
```


## Resources

- [Readonly Properties](https://www.php.net/manual/en/language.oop5.properties.php#language.oop5.properties.readonly-properties) — Official documentation for readonly properties

---

> 📘 *This lesson is part of the [Modern PHP 8.x: Latest Language Features](https://stanza.dev/courses/php-modern-features) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*