---
source_course: "php-oop-mastery"
source_lesson: "php-oop-mastery-strategy-pattern"
---

# Strategy Pattern

## Introduction
The Strategy pattern defines a family of algorithms, encapsulates each one, and makes them interchangeable. It lets you change an object's behavior at runtime without modifying the object itself.

## Key Concepts
- **Strategy Interface**: Defines the contract that all algorithms must follow.
- **Concrete Strategies**: Different implementations of the algorithm.
- **Context**: The object that uses a strategy and delegates behavior to it.
- **Runtime Swapping**: Strategies can be changed at runtime without modifying client code.

## Real World Context
An e-commerce platform needs to support multiple discount strategies: percentage off, fixed amount off, buy-one-get-one, and seasonal promotions. Rather than hardcoding discount logic with if/else chains, each discount type is a strategy that can be applied to any order.

## Deep Dive

### Basic Implementation

```php
<?php
interface PaymentStrategy {
    public function pay(float $amount): bool;
    public function getName(): string;
}

class CreditCardPayment implements PaymentStrategy {
    public function __construct(
        private string $cardNumber,
        private string $cvv
    ) {}

    public function pay(float $amount): bool {
        echo "Charging \${$amount} to credit card\n";
        return true;
    }

    public function getName(): string { return 'Credit Card'; }
}

class PayPalPayment implements PaymentStrategy {
    public function __construct(private string $email) {}

    public function pay(float $amount): bool {
        echo "Transferring \${$amount} via PayPal\n";
        return true;
    }

    public function getName(): string { return 'PayPal'; }
}

class ShoppingCart {
    private array $items = [];
    private ?PaymentStrategy $paymentMethod = null;

    public function addItem(string $name, float $price): void {
        $this->items[] = ['name' => $name, 'price' => $price];
    }

    public function setPaymentMethod(PaymentStrategy $method): void {
        $this->paymentMethod = $method;
    }

    public function checkout(): bool {
        if ($this->paymentMethod === null) {
            throw new RuntimeException('No payment method');
        }
        $total = array_sum(array_column($this->items, 'price'));
        return $this->paymentMethod->pay($total);
    }
}

$cart = new ShoppingCart();
$cart->addItem('Widget', 29.99);
$cart->setPaymentMethod(new CreditCardPayment('4111...', '123'));
$cart->checkout();

// Switch strategy at runtime
$cart->setPaymentMethod(new PayPalPayment('user@example.com'));
$cart->checkout();
```

The cart does not know or care which payment method is used. Strategies are interchangeable.

### Discount Strategy Example

```php
<?php
interface DiscountStrategy {
    public function calculate(float $total): float;
    public function getDescription(): string;
}

class NoDiscount implements DiscountStrategy {
    public function calculate(float $total): float { return $total; }
    public function getDescription(): string { return 'No discount'; }
}

class PercentageDiscount implements DiscountStrategy {
    public function __construct(private float $percent) {}

    public function calculate(float $total): float {
        return $total * (1 - $this->percent / 100);
    }

    public function getDescription(): string {
        return "{$this->percent}% off";
    }
}

class FixedDiscount implements DiscountStrategy {
    public function __construct(private float $amount) {}

    public function calculate(float $total): float {
        return max(0, $total - $this->amount);
    }

    public function getDescription(): string {
        return "\${$this->amount} off";
    }
}

class Order {
    private DiscountStrategy $discount;

    public function __construct(private float $subtotal) {
        $this->discount = new NoDiscount();
    }

    public function applyDiscount(DiscountStrategy $discount): void {
        $this->discount = $discount;
    }

    public function getTotal(): float {
        return $this->discount->calculate($this->subtotal);
    }
}
```

New discount types can be added without modifying the `Order` class.

## Common Pitfalls
1. **Too many strategies** — If you have dozens of nearly identical strategies, consider parameterizing a single strategy instead.
2. **Strategy selection scattered** — Centralize strategy selection (via factory) rather than choosing strategies in multiple places.

## Best Practices
1. **Combine with Factory** — Use a factory to create the right strategy based on configuration or user input.
2. **Keep strategies stateless when possible** — Stateless strategies are easier to share, cache, and test.

## Summary
- Strategy pattern encapsulates interchangeable algorithms behind a common interface.
- Strategies can be swapped at runtime without changing client code.
- Combine with Factory pattern for clean strategy selection.
- Keep strategies focused on a single algorithm.

## Code Examples

**Sorting strategy pattern**

```php
<?php
declare(strict_types=1);

interface SortStrategy {
    public function sort(array &$data): void;
    public function getName(): string;
}

class BubbleSort implements SortStrategy {
    public function sort(array &$data): void { sort($data); }
    public function getName(): string { return 'Bubble Sort'; }
}

class QuickSort implements SortStrategy {
    public function sort(array &$data): void { sort($data); }
    public function getName(): string { return 'Quick Sort'; }
}

class DataProcessor {
    public function __construct(private SortStrategy $strategy) {}

    public function setStrategy(SortStrategy $strategy): void {
        $this->strategy = $strategy;
    }

    public function process(array $data): array {
        $this->strategy->sort($data);
        return $data;
    }
}

$processor = new DataProcessor(new QuickSort());
$result = $processor->process([3, 1, 4, 1, 5]);
?>
```


## Resources

- [Strategy Pattern](https://refactoring.guru/design-patterns/strategy/php/example) — Strategy pattern examples

---

> 📘 *This lesson is part of the [Object-Oriented PHP Mastery](https://stanza.dev/courses/php-oop-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*