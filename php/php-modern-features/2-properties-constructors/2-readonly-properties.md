---
source_course: "php-modern-features"
source_lesson: "php-modern-features-readonly-properties"
---

# Readonly Properties and Classes

## Introduction
Readonly properties, introduced in PHP 8.1, can only be initialized once and then become immutable. PHP 8.2 extended this with readonly classes where all properties are implicitly readonly. Together, they make value objects and DTOs first-class citizens in PHP.

## Key Concepts
- **Readonly Property**: A property that can only be assigned once, after which any modification throws an `Error`.
- **Readonly Class**: A class where all properties are implicitly readonly (PHP 8.2+).
- **Clone with Override**: PHP 8.5 introduces `clone()` with property overrides, enabling the "wither" pattern for readonly classes.

## Real World Context
Immutability is a cornerstone of modern application design. Readonly properties guarantee that value objects, DTOs, and configuration instances cannot be accidentally modified after creation, eliminating an entire class of bugs.

## Deep Dive

### Readonly Properties (PHP 8.1+)

A readonly property can only be assigned once:

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

Once set, the property is permanently locked.

### Readonly Property Rules

Readonly properties have specific constraints:

```php
<?php
class Example {
    // Must have a type declaration
    public readonly string $valid;
    
    // Cannot have a default value outside constructor promotion
    // public readonly string $withDefault = 'nope';  // Error!
    
    // OK in constructor promotion
    public function __construct(
        public readonly string $promoted = 'default'
    ) {}
}
```

The property must be typed and can only be assigned from within the declaring class.

### Readonly Classes (PHP 8.2+)

Mark an entire class as readonly to make all properties immutable:

```php
<?php
readonly class Point {
    public function __construct(
        public float $x,
        public float $y
    ) {}
}

$point = new Point(1.0, 2.0);
$point->x = 3.0;  // Error! Cannot modify
```

This is equivalent to declaring every property as `public readonly`, but more concise. All properties in a readonly class must be typed, and static properties are not allowed.

### Clone with Property Overrides (PHP 8.5)

PHP 8.5 introduces `clone()` with property overrides, solving the biggest pain point of readonly classes — creating modified copies:

```php
<?php
readonly class Color {
    public function __construct(
        public int $red,
        public int $green,
        public int $blue,
        public int $alpha = 255
    ) {}
    
    public function withAlpha(int $alpha): self {
        return clone($this, ['alpha' => $alpha]);
    }
}

$red = new Color(255, 0, 0);
$semiTransparent = $red->withAlpha(128);
// $semiTransparent->alpha === 128, $semiTransparent->red === 255
```

Before PHP 8.5, creating a modified copy of a readonly object required manually passing every property to the constructor. The `clone()` function copies all properties and overrides only the ones specified via the second argument.

## Common Pitfalls
1. **Trying to modify readonly properties in subclasses** — Readonly properties cannot be modified even by child classes. Once set in the parent constructor, they are permanently locked.
2. **Forgetting the type declaration** — `public readonly $untyped` is a compile error. Every readonly property must have a type.

## Best Practices
1. **Use readonly classes for value objects and DTOs** — If a class represents data that should not change after creation, make it `readonly`.
2. **Use the wither pattern with clone** — In PHP 8.5, use `clone()` with property overrides to create modified copies. In earlier versions, provide `withX()` methods that call `new self(...)`.

## Summary
- Readonly properties can be assigned once and never modified.
- Readonly classes make all properties implicitly readonly.
- PHP 8.5's `clone()` function with property overrides enables the wither pattern for readonly objects.
- Readonly properties must have a type declaration.
- Use them for value objects, DTOs, and any data that should be immutable.

## Code Examples

**Immutable Money value object using readonly class — every operation returns a new instance**

```php
<?php
declare(strict_types=1);

readonly class Money {
    public function __construct(
        public int $amount,
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

$price = new Money(1999, 'USD');
$tax = $price->multiply(0.1);
$total = $price->add($tax);
echo $total->format();  // USD 21.98
?>
```


## Resources

- [Readonly Properties](https://www.php.net/manual/en/language.oop5.properties.php#language.oop5.properties.readonly-properties) — Official documentation for readonly properties

---

> 📘 *This lesson is part of the [Modern PHP 8.x: Latest Language Features](https://stanza.dev/courses/php-modern-features) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*