---
source_course: "php-oop-mastery"
source_lesson: "php-oop-mastery-strategy-pattern"
---

# Strategy Pattern

The Strategy pattern defines a family of algorithms, encapsulates each one, and makes them interchangeable.

## Basic Implementation

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
    
    public function getName(): string {
        return 'Credit Card';
    }
}

class PayPalPayment implements PaymentStrategy {
    public function __construct(
        private string $email
    ) {}
    
    public function pay(float $amount): bool {
        echo "Transferring \${$amount} via PayPal\n";
        return true;
    }
    
    public function getName(): string {
        return 'PayPal';
    }
}

class CryptoPayment implements PaymentStrategy {
    public function __construct(
        private string $walletAddress
    ) {}
    
    public function pay(float $amount): bool {
        echo "Sending \${$amount} in crypto\n";
        return true;
    }
    
    public function getName(): string {
        return 'Cryptocurrency';
    }
}

// Context
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
            throw new RuntimeException('No payment method selected');
        }
        
        $total = array_sum(array_column($this->items, 'price'));
        echo "Paying with: {$this->paymentMethod->getName()}\n";
        return $this->paymentMethod->pay($total);
    }
}

// Usage
$cart = new ShoppingCart();
$cart->addItem('Widget', 29.99);
$cart->addItem('Gadget', 49.99);

// Choose strategy at runtime
$cart->setPaymentMethod(new CreditCardPayment('4111...', '123'));
$cart->checkout();

// Or switch to different strategy
$cart->setPaymentMethod(new PayPalPayment('user@example.com'));
$cart->checkout();
```

## Sorting Strategy Example

```php
<?php
interface SortStrategy {
    public function sort(array $data): array;
}

class QuickSort implements SortStrategy {
    public function sort(array $data): array {
        sort($data);  // PHP's built-in quicksort
        return $data;
    }
}

class MergeSort implements SortStrategy {
    public function sort(array $data): array {
        // Merge sort implementation
        return $this->mergeSort($data);
    }
    
    private function mergeSort(array $arr): array {
        // ...
    }
}

class Sorter {
    public function __construct(
        private SortStrategy $strategy
    ) {}
    
    public function setStrategy(SortStrategy $strategy): void {
        $this->strategy = $strategy;
    }
    
    public function sort(array $data): array {
        return $this->strategy->sort($data);
    }
}
```

## Code Examples

**Discount calculation using Strategy pattern**

```php
<?php
declare(strict_types=1);

// Discount strategy pattern
interface DiscountStrategy {
    public function calculate(float $total): float;
    public function getDescription(): string;
}

class NoDiscount implements DiscountStrategy {
    public function calculate(float $total): float {
        return $total;
    }
    
    public function getDescription(): string {
        return 'No discount applied';
    }
}

class PercentageDiscount implements DiscountStrategy {
    public function __construct(
        private float $percent
    ) {}
    
    public function calculate(float $total): float {
        return $total * (1 - $this->percent / 100);
    }
    
    public function getDescription(): string {
        return "{$this->percent}% off";
    }
}

class FixedDiscount implements DiscountStrategy {
    public function __construct(
        private float $amount
    ) {}
    
    public function calculate(float $total): float {
        return max(0, $total - $this->amount);
    }
    
    public function getDescription(): string {
        return "\${$this->amount} off";
    }
}

class Order {
    private DiscountStrategy $discount;
    
    public function __construct(
        private float $subtotal
    ) {
        $this->discount = new NoDiscount();
    }
    
    public function applyDiscount(DiscountStrategy $discount): void {
        $this->discount = $discount;
    }
    
    public function getTotal(): float {
        return $this->discount->calculate($this->subtotal);
    }
    
    public function getSummary(): string {
        return sprintf(
            "Subtotal: \$%.2f\nDiscount: %s\nTotal: \$%.2f",
            $this->subtotal,
            $this->discount->getDescription(),
            $this->getTotal()
        );
    }
}

$order = new Order(100.00);
echo $order->getSummary() . "\n\n";

$order->applyDiscount(new PercentageDiscount(20));
echo $order->getSummary();
?>
```


---

> 📘 *This lesson is part of the [Object-Oriented PHP Mastery](https://stanza.dev/courses/php-oop-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*