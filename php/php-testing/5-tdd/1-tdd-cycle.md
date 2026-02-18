---
source_course: "php-testing"
source_lesson: "php-testing-tdd-cycle"
---

# The TDD Cycle

Test-Driven Development follows a simple cycle: Red → Green → Refactor.

## The Cycle

```
┌─────────────────────────────────────────────────────┐
│                                                     │
│    🔴 RED              🟢 GREEN           🔵 REFACTOR │
│    Write failing  →   Make it pass  →   Clean up   │
│    test                                             │
│         ↑                                    │      │
│         └────────────────────────────────────┘      │
│                                                     │
└─────────────────────────────────────────────────────┘
```

## 1. RED: Write a Failing Test

```php
<?php
// Start with what you WANT the code to do
class ShoppingCartTest extends TestCase
{
    public function testNewCartIsEmpty(): void
    {
        $cart = new ShoppingCart();
        
        $this->assertEquals(0, $cart->getItemCount());
        $this->assertEquals(0.0, $cart->getTotal());
    }
}

// Run test: FAIL (ShoppingCart class doesn't exist)
```

## 2. GREEN: Make It Pass (Minimum Code)

```php
<?php
class ShoppingCart
{
    public function getItemCount(): int
    {
        return 0;
    }
    
    public function getTotal(): float
    {
        return 0.0;
    }
}

// Run test: PASS
```

## 3. REFACTOR: Improve Without Breaking

```php
<?php
class ShoppingCart
{
    private array $items = [];
    
    public function getItemCount(): int
    {
        return count($this->items);
    }
    
    public function getTotal(): float
    {
        return array_sum(array_column($this->items, 'subtotal'));
    }
}

// Run test: Still PASS
```

## Continue the Cycle

```php
<?php
// New test (RED)
public function testAddItem(): void
{
    $cart = new ShoppingCart();
    
    $cart->addItem(new Product('Widget', 10.00), quantity: 2);
    
    $this->assertEquals(1, $cart->getItemCount());
    $this->assertEquals(20.00, $cart->getTotal());
}

// Implement (GREEN)
public function addItem(Product $product, int $quantity): void
{
    $this->items[] = [
        'product' => $product,
        'quantity' => $quantity,
        'subtotal' => $product->price * $quantity,
    ];
}

// Refactor as needed...
```

## TDD Benefits

- **Design**: Tests drive clean, modular design
- **Documentation**: Tests explain expected behavior
- **Confidence**: Refactor without fear
- **Focus**: Work on one thing at a time
- **Coverage**: High test coverage by default

---

> 📘 *This lesson is part of the [PHP Testing & Quality Assurance](https://stanza.dev/courses/php-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*