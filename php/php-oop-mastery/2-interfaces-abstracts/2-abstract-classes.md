---
source_course: "php-oop-mastery"
source_lesson: "php-oop-mastery-abstract-classes"
---

# Abstract Classes

Abstract classes provide partial implementation while requiring subclasses to implement specific methods.

## Basic Abstract Class

```php
<?php
abstract class Shape {
    public function __construct(
        protected string $color
    ) {}
    
    // Abstract method - must be implemented
    abstract public function getArea(): float;
    
    // Concrete method - inherited as-is
    public function getColor(): string {
        return $this->color;
    }
    
    // Concrete method using abstract method
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

## Template Method Pattern

```php
<?php
abstract class DataExporter {
    // Template method - defines the algorithm
    final public function export(array $data): string {
        $this->validate($data);
        $formatted = $this->format($data);
        return $this->output($formatted);
    }
    
    // Concrete step
    protected function validate(array $data): void {
        if (empty($data)) {
            throw new InvalidArgumentException('Data cannot be empty');
        }
    }
    
    // Abstract steps - subclasses must implement
    abstract protected function format(array $data): string;
    abstract protected function output(string $content): string;
}

class CsvExporter extends DataExporter {
    protected function format(array $data): string {
        $lines = [];
        foreach ($data as $row) {
            $lines[] = implode(',', $row);
        }
        return implode("\n", $lines);
    }
    
    protected function output(string $content): string {
        return $content;  // Return as-is
    }
}

class JsonExporter extends DataExporter {
    protected function format(array $data): string {
        return json_encode($data, JSON_PRETTY_PRINT);
    }
    
    protected function output(string $content): string {
        return $content;
    }
}
```

## Abstract vs Interface

| Feature | Interface | Abstract Class |
|---------|-----------|----------------|
| Methods | Only signatures | Both abstract and concrete |
| Properties | Constants only | Any properties |
| Multiple | Yes | No (single inheritance) |
| Constructor | No | Yes |
| Visibility | Public only | Any visibility |
| Use when | Defining a contract | Sharing implementation |

## When to Use Abstract Classes

```php
<?php
// Use abstract class when:
// 1. You have shared implementation
// 2. You need protected members
// 3. You want a template method pattern

abstract class Repository {
    public function __construct(
        protected PDO $pdo  // Shared dependency
    ) {}
    
    // Shared implementation
    protected function execute(string $sql, array $params = []): PDOStatement {
        $stmt = $this->pdo->prepare($sql);
        $stmt->execute($params);
        return $stmt;
    }
    
    // Subclasses define their entity
    abstract protected function getTableName(): string;
    abstract protected function hydrate(array $row): object;
}
```

## Code Examples

**Abstract payment processor with template method pattern**

```php
<?php
declare(strict_types=1);

// Abstract payment processor with template method
abstract class PaymentProcessor {
    abstract protected function validatePayment(float $amount): void;
    abstract protected function processTransaction(float $amount): string;
    abstract protected function sendReceipt(string $transactionId): void;
    
    // Template method
    final public function processPayment(float $amount): string {
        $this->validatePayment($amount);
        $transactionId = $this->processTransaction($amount);
        $this->sendReceipt($transactionId);
        return $transactionId;
    }
    
    // Hook method - can be overridden
    protected function logTransaction(string $id): void {
        echo "Transaction logged: $id\n";
    }
}

class StripeProcessor extends PaymentProcessor {
    protected function validatePayment(float $amount): void {
        if ($amount < 0.50) {
            throw new InvalidArgumentException('Minimum amount is $0.50');
        }
    }
    
    protected function processTransaction(float $amount): string {
        // Stripe API call
        return 'stripe_' . bin2hex(random_bytes(8));
    }
    
    protected function sendReceipt(string $transactionId): void {
        echo "Stripe receipt sent for: $transactionId\n";
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