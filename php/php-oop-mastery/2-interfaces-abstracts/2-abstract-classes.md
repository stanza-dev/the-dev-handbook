---
source_course: "php-oop-mastery"
source_lesson: "php-oop-mastery-abstract-classes"
---

# Abstract Classes

## Introduction
Abstract classes provide partial implementation while requiring subclasses to implement specific methods. They sit between interfaces (pure contracts) and concrete classes (full implementations).

## Key Concepts
- **Abstract Methods**: Methods declared without a body that subclasses must implement.
- **Concrete Methods**: Fully implemented methods that subclasses inherit directly.
- **Template Method Pattern**: An abstract class defines the skeleton of an algorithm, deferring steps to subclasses.
- **Cannot Instantiate**: You cannot create an instance of an abstract class directly.

## Real World Context
Consider a data export system. Every exporter validates, formats, and outputs data. Validation and output are the same, but formatting differs for CSV, JSON, and XML. An abstract `DataExporter` implements shared steps and leaves format-specific logic abstract.

## Deep Dive

### Basic Abstract Class

```php
<?php
abstract class Shape {
    public function __construct(protected string $color) {}

    abstract public function getArea(): float;

    public function getColor(): string {
        return $this->color;
    }

    public function describe(): string {
        return "A {$this->color} shape with area " . $this->getArea();
    }
}

class Rectangle extends Shape {
    public function __construct(
        string $color,
        private float $width,
        private float $height
    ) {
        parent::__construct($color);
    }

    public function getArea(): float {
        return $this->width * $this->height;
    }
}

class Circle extends Shape {
    public function __construct(
        string $color,
        private float $radius
    ) {
        parent::__construct($color);
    }

    public function getArea(): float {
        return pi() * $this->radius ** 2;
    }
}
```

The `describe()` method works for all shapes because it calls the abstract `getArea()` that each shape implements.

### Template Method Pattern

```php
<?php
abstract class DataExporter {
    final public function export(array $data): string {
        $this->validate($data);
        $formatted = $this->format($data);
        return $this->output($formatted);
    }

    protected function validate(array $data): void {
        if (empty($data)) {
            throw new InvalidArgumentException('Data cannot be empty');
        }
    }

    abstract protected function format(array $data): string;
    abstract protected function output(string $content): string;
}

class CsvExporter extends DataExporter {
    protected function format(array $data): string {
        $lines = [];
        foreach ($data as $row) { $lines[] = implode(',', $row); }
        return implode("\n", $lines);
    }

    protected function output(string $content): string {
        return $content;
    }
}
```

The `final` keyword prevents subclasses from overriding the algorithm structure.

### Abstract vs Interface

| Feature | Interface | Abstract Class |
|---------|-----------|----------------|
| Methods | Only signatures | Both abstract and concrete |
| Properties | Constants only | Any properties |
| Multiple | Yes | No (single inheritance) |
| Constructor | No | Yes |
| Visibility | Public only | Any visibility |
| Use when | Defining a contract | Sharing implementation |

### When to Use Abstract Classes

```php
<?php
abstract class Repository {
    public function __construct(protected PDO $pdo) {}

    protected function execute(string $sql, array $params = []): PDOStatement {
        $stmt = $this->pdo->prepare($sql);
        $stmt->execute($params);
        return $stmt;
    }

    abstract protected function getTableName(): string;
    abstract protected function hydrate(array $row): object;
}
```

Use abstract classes when subclasses share implementation and need protected members.

## Common Pitfalls
1. **Using abstract classes when interfaces suffice** — If you only need a contract without shared implementation, use an interface.
2. **Forgetting parent constructor** — When a subclass defines its own constructor, it must call `parent::__construct()`.

## Best Practices
1. **Use final on template methods** — Mark template methods as `final` to prevent subclasses from breaking the algorithm.
2. **Combine with interfaces** — An abstract class can implement an interface, giving you both a contract and shared implementation.

## Summary
- Abstract classes provide partial implementation with abstract and concrete methods.
- They cannot be instantiated directly and must be extended.
- The Template Method pattern leverages abstract classes for algorithm skeletons.
- Use abstract classes for sharing implementation; use interfaces for contracts only.

## Code Examples

**Abstract payment processor with template method**

```php
<?php
declare(strict_types=1);

abstract class PaymentProcessor {
    abstract protected function validatePayment(float $amount): void;
    abstract protected function processTransaction(float $amount): string;
    abstract protected function sendReceipt(string $transactionId): void;

    final public function processPayment(float $amount): string {
        $this->validatePayment($amount);
        $transactionId = $this->processTransaction($amount);
        $this->sendReceipt($transactionId);
        return $transactionId;
    }
}

class StripeProcessor extends PaymentProcessor {
    protected function validatePayment(float $amount): void {
        if ($amount < 0.50) {
            throw new InvalidArgumentException('Minimum is $0.50');
        }
    }

    protected function processTransaction(float $amount): string {
        return 'stripe_' . bin2hex(random_bytes(8));
    }

    protected function sendReceipt(string $transactionId): void {
        echo "Stripe receipt: $transactionId\n";
    }
}

$processor = new StripeProcessor();
$txId = $processor->processPayment(99.99);
?>
```


## Resources

- [Class Abstraction](https://www.php.net/manual/en/language.oop5.abstract.php) — PHP abstract classes documentation

---

> 📘 *This lesson is part of the [Object-Oriented PHP Mastery](https://stanza.dev/courses/php-oop-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*