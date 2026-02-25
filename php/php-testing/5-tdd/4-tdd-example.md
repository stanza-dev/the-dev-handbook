---
source_course: "php-testing"
source_lesson: "php-testing-tdd-example"
---

# TDD in Practice: Building a Feature

## Introduction
Theory only takes you so far. This lesson walks through a complete, realistic TDD session: building a shopping cart with discount rules. You will see the full Red-Green-Refactor cycle repeat across multiple iterations, each one adding a new behavior.

## Key Concepts
- **Iterative Design**: TDD builds features through small, repeated cycles rather than one big implementation.
- **Emergent Design**: The code's architecture emerges from the tests rather than being planned upfront.
- **Test List**: A checklist of behaviors to test, written before coding begins, guiding the order of TDD cycles.

## Real World Context
Building a shopping cart with discount logic is a common real-world scenario. Discounts have edge cases: applying multiple discounts, handling zero quantities, percentage versus fixed discounts. TDD ensures each rule is tested individually, making the final system reliable and easy to modify when business rules change.

## Deep Dive

### Step 0: Write a Test List
Before touching code, list the behaviors you want to build. This is your roadmap.

```text
- [ ] Cart starts empty with zero total
- [ ] Adding a product updates the total
- [ ] Adding multiple products sums their prices
- [ ] Applying a percentage discount reduces the total
- [ ] Discount cannot make total negative
```

Now tackle them one at a time.

### Iteration 1: Empty Cart (Red-Green-Refactor)

**Red:** Write the first failing test.

```php
<?php
use PHPUnit\Framework\TestCase;

class ShoppingCartTest extends TestCase
{
    public function testNewCartHasZeroTotal(): void
    {
        $cart = new ShoppingCart();

        $this->assertSame(0.0, $cart->getTotal());
    }
}
```

This fails because `ShoppingCart` does not exist.

**Green:** Write the minimum to pass.

```php
<?php
class ShoppingCart
{
    public function getTotal(): float
    {
        return 0.0;
    }
}
```

Test passes. No refactoring needed yet.

### Iteration 2: Adding a Product (Red-Green-Refactor)

**Red:** Add a test for adding a product.

```php
<?php
public function testAddingProductUpdatesTotal(): void
{
    $cart = new ShoppingCart();

    $cart->addProduct('PHP Book', 29.99);

    $this->assertSame(29.99, $cart->getTotal());
}
```

This fails because `addProduct` does not exist.

**Green:** Implement `addProduct`.

```php
<?php
class ShoppingCart
{
    private array $items = [];

    public function addProduct(string $name, float $price): void
    {
        $this->items[] = ['name' => $name, 'price' => $price];
    }

    public function getTotal(): float
    {
        $total = 0.0;
        foreach ($this->items as $item) {
            $total += $item['price'];
        }
        return $total;
    }
}
```

Both tests pass.

**Refactor:** The array structure works, but using a value object would be cleaner. However, since only two tests exist, this refactoring can wait.

### Iteration 3: Multiple Products

**Red:** Test adding two products.

```php
<?php
public function testMultipleProductsSumCorrectly(): void
{
    $cart = new ShoppingCart();

    $cart->addProduct('PHP Book', 29.99);
    $cart->addProduct('Testing Guide', 19.99);

    $this->assertSame(49.98, $cart->getTotal());
}
```

**Green:** This already passes with the existing implementation. If a new test passes immediately, that means the behavior was already covered. Move on.

### Iteration 4: Percentage Discount (Red-Green-Refactor)

**Red:** Test applying a discount.

```php
<?php
public function testPercentageDiscountReducesTotal(): void
{
    $cart = new ShoppingCart();
    $cart->addProduct('PHP Book', 100.00);

    $cart->applyDiscount(10); // 10% off

    $this->assertSame(90.0, $cart->getTotal());
}
```

Fails: `applyDiscount` does not exist.

**Green:** Implement the discount.

```php
<?php
class ShoppingCart
{
    private array $items = [];
    private float $discountPercent = 0.0;

    public function addProduct(string $name, float $price): void
    {
        $this->items[] = ['name' => $name, 'price' => $price];
    }

    public function applyDiscount(float $percent): void
    {
        $this->discountPercent = $percent;
    }

    public function getTotal(): float
    {
        $subtotal = 0.0;
        foreach ($this->items as $item) {
            $subtotal += $item['price'];
        }

        $discount = $subtotal * ($this->discountPercent / 100);

        return $subtotal - $discount;
    }
}
```

All four tests pass.

**Refactor:** Extract the subtotal calculation into its own method for clarity.

```php
<?php
class ShoppingCart
{
    private array $items = [];
    private float $discountPercent = 0.0;

    public function addProduct(string $name, float $price): void
    {
        $this->items[] = ['name' => $name, 'price' => $price];
    }

    public function applyDiscount(float $percent): void
    {
        $this->discountPercent = $percent;
    }

    public function getTotal(): float
    {
        $subtotal = $this->getSubtotal();
        $discount = $subtotal * ($this->discountPercent / 100);

        return $subtotal - $discount;
    }

    private function getSubtotal(): float
    {
        $subtotal = 0.0;
        foreach ($this->items as $item) {
            $subtotal += $item['price'];
        }
        return $subtotal;
    }
}
```

All tests still pass after refactoring. The design emerged naturally from the tests.

### Iteration 5: Discount Floor (Red-Green-Refactor)

**Red:** Ensure discount cannot make total negative.

```php
<?php
public function testDiscountCannotMakeTotalNegative(): void
{
    $cart = new ShoppingCart();
    $cart->addProduct('Sticker', 5.00);

    $cart->applyDiscount(200); // 200% discount

    $this->assertSame(0.0, $cart->getTotal());
}
```

**Green:** Add a floor to `getTotal`.

```php
<?php
public function getTotal(): float
{
    $subtotal = $this->getSubtotal();
    $discount = $subtotal * ($this->discountPercent / 100);

    return max(0.0, $subtotal - $discount);
}
```

All five tests pass. The shopping cart is complete, built entirely through TDD.

## Common Pitfalls
1. **Trying to build everything in one iteration** — TDD works through many small cycles, not one large one. Each cycle adds one behavior. Rushing to add discounts, taxes, and shipping in a single step defeats the purpose.
2. **Not refactoring when you should** — If you skip the Refactor phase, your code accumulates technical debt even with tests. The tests give you a safety net to refactor fearlessly — use it.

## Best Practices
1. **Start with a test list** — Before coding, write down the behaviors you plan to test. This gives direction to your TDD session and prevents you from wandering.
2. **Keep the cycle under five minutes** — Each Red-Green-Refactor iteration should be short. If you are spending more than five minutes in the Green phase, your step is too large. Break it down further.

## Summary
- A complete TDD session iterates through Red-Green-Refactor many times, each cycle adding one behavior.
- Start with a test list to plan your order of attack.
- The design of the code emerges from the tests, rather than being planned in advance.
- Refactoring after each Green phase keeps the code clean as it grows.
- Each iteration should take no more than a few minutes, keeping the feedback loop tight.

## Code Examples

**Complete shopping cart built with TDD: five tests and the final implementation showing emergent design**

```php
<?php
declare(strict_types=1);

use PHPUnit\Framework\TestCase;

// Final test suite after five TDD iterations
class ShoppingCartTest extends TestCase
{
    public function testNewCartHasZeroTotal(): void
    {
        $cart = new ShoppingCart();
        $this->assertSame(0.0, $cart->getTotal());
    }

    public function testAddingProductUpdatesTotal(): void
    {
        $cart = new ShoppingCart();
        $cart->addProduct('PHP Book', 29.99);
        $this->assertSame(29.99, $cart->getTotal());
    }

    public function testMultipleProductsSumCorrectly(): void
    {
        $cart = new ShoppingCart();
        $cart->addProduct('PHP Book', 29.99);
        $cart->addProduct('Testing Guide', 19.99);
        $this->assertSame(49.98, $cart->getTotal());
    }

    public function testPercentageDiscountReducesTotal(): void
    {
        $cart = new ShoppingCart();
        $cart->addProduct('PHP Book', 100.00);
        $cart->applyDiscount(10);
        $this->assertSame(90.0, $cart->getTotal());
    }

    public function testDiscountCannotMakeTotalNegative(): void
    {
        $cart = new ShoppingCart();
        $cart->addProduct('Sticker', 5.00);
        $cart->applyDiscount(200);
        $this->assertSame(0.0, $cart->getTotal());
    }
}

// Final ShoppingCart after all iterations
class ShoppingCart
{
    private array $items = [];
    private float $discountPercent = 0.0;

    public function addProduct(string $name, float $price): void
    {
        $this->items[] = ['name' => $name, 'price' => $price];
    }

    public function applyDiscount(float $percent): void
    {
        $this->discountPercent = $percent;
    }

    public function getTotal(): float
    {
        $subtotal = $this->getSubtotal();
        $discount = $subtotal * ($this->discountPercent / 100);
        return max(0.0, $subtotal - $discount);
    }

    private function getSubtotal(): float
    {
        $total = 0.0;
        foreach ($this->items as $item) {
            $total += $item['price'];
        }
        return $total;
    }
}
```


## Resources

- [PHPUnit Documentation - Test Fixtures](https://docs.phpunit.de/en/12.0/fixtures.html) — Setting up shared test fixtures with setUp and tearDown for iterative TDD
- [PHP Classes and Objects](https://www.php.net/manual/en/language.oop5.php) — PHP OOP reference for building classes driven by TDD

---

> 📘 *This lesson is part of the [PHP Testing & Quality Assurance](https://stanza.dev/courses/php-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*