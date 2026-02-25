---
source_course: "php-testing"
source_lesson: "php-testing-tdd-red-phase"
---

# Writing the First Failing Test

## Introduction
The Red phase is the most important step in TDD. By writing a test before any production code, you establish a clear contract for what the code must do. This lesson teaches you how to craft effective failing tests and resist the urge to write the implementation first.

## Key Concepts
- **Failing Test**: A test that does not pass because the feature under test has not been implemented yet.
- **Test Contract**: The test serves as a specification, defining the expected behavior of the code before it exists.
- **Meaningful Failure**: The test should fail for the right reason (missing behavior), not because of a typo or misconfiguration.

## Real World Context
Every bug you have ever fixed in production could have been prevented by a failing test written before the fix. The Red phase disciplines you to think about expected behavior upfront. In teams, a failing test written from a bug report becomes an executable specification that proves the bug is fixed and can never silently return.

## Deep Dive

### Start With the Simplest Case
When beginning a new feature, pick the simplest, most obvious behavior to test first. For a shopping cart, that might be "an empty cart has zero items."

Here is how you write that first test before any `ShoppingCart` class exists.

```php
<?php
use PHPUnit\Framework\TestCase;

class ShoppingCartTest extends TestCase
{
    public function testNewCartIsEmpty(): void
    {
        $cart = new ShoppingCart();

        $this->assertSame(0, $cart->getItemCount());
    }
}
```

Running this produces a clear error: the `ShoppingCart` class does not exist. That failure is exactly what you want to see.

### Name Tests as Specifications
Good test names read like requirements. They describe what the system should do, not how the test works.

```php
<?php
// Good: reads like a specification
public function testEmptyCartHasZeroTotal(): void { /* ... */ }
public function testAddingItemIncreasesCount(): void { /* ... */ }
public function testRemovingLastItemMakesCartEmpty(): void { /* ... */ }

// Bad: describes implementation, not behavior
public function testArray(): void { /* ... */ }
public function testFunction(): void { /* ... */ }
```

Descriptive names turn your test suite into living documentation that any developer can read.

### Test One Behavior at a Time
Each test should verify one specific behavior. If a test has multiple unrelated assertions, split it into separate tests.

```php
<?php
// Bad: testing two unrelated behaviors
public function testCart(): void
{
    $cart = new ShoppingCart();
    $this->assertSame(0, $cart->getItemCount());
    $cart->addItem(new Product('Widget', 9.99));
    $this->assertSame(9.99, $cart->getTotal());
}

// Good: separate tests for separate behaviors
public function testNewCartIsEmpty(): void
{
    $cart = new ShoppingCart();
    $this->assertSame(0, $cart->getItemCount());
}

public function testAddingItemUpdatesTotal(): void
{
    $cart = new ShoppingCart();
    $cart->addItem(new Product('Widget', 9.99));
    $this->assertSame(9.99, $cart->getTotal());
}
```

Splitting tests makes failures easier to diagnose. When a test fails, you immediately know which behavior broke.

### Resist the Urge to Implement First
The most common mistake beginners make is thinking about the implementation before writing the test. TDD reverses this: the test defines what you need, and the implementation serves the test.

If you find yourself thinking "I need a method that does X, then Y, then Z," stop. Instead think: "What should the result look like? What test would verify that result?"

## Common Pitfalls
1. **Writing tests that pass immediately** — If your new test passes without writing any production code, it is not testing new behavior. Either the behavior already exists or the test is flawed. Always verify you see the red bar first.
2. **Testing too much in one test** — A test with five assertions is really five tests crammed together. When it fails, you cannot tell which behavior broke. Keep one logical assertion per test.

## Best Practices
1. **Use descriptive test names** — Name your tests as behavioral specifications: `testGuestUserCannotCheckout`, not `testCheckout`. Good names make test failures self-explanatory.
2. **Follow the Arrange-Act-Assert pattern** — Structure every test into three sections: set up the scenario (Arrange), perform the action (Act), and verify the outcome (Assert). This keeps tests readable and consistent.

## Summary
- Always start with a test that fails for the right reason.
- Pick the simplest behavior to test first and build complexity incrementally.
- Name tests like specifications so they serve as living documentation.
- Test one behavior per test method to keep failures diagnostic.
- Resist the urge to write production code before the test exists.

## Code Examples

**A series of failing tests for a ShoppingCart, written before any implementation exists, following the Arrange-Act-Assert pattern**

```php
<?php
declare(strict_types=1);

namespace Tests\Unit;

use PHPUnit\Framework\TestCase;

// Writing failing tests for a ShoppingCart that does not exist yet
class ShoppingCartTest extends TestCase
{
    public function testNewCartIsEmpty(): void
    {
        // Arrange & Act
        $cart = new \App\ShoppingCart();

        // Assert: a new cart should have no items
        $this->assertSame(0, $cart->getItemCount());
        $this->assertSame(0.0, $cart->getTotal());
    }

    public function testAddingItemIncreasesCount(): void
    {
        // Arrange
        $cart = new \App\ShoppingCart();
        $product = new \App\Product('PHP Book', 29.99);

        // Act
        $cart->addItem($product);

        // Assert
        $this->assertSame(1, $cart->getItemCount());
    }

    public function testTotalReflectsAddedItems(): void
    {
        // Arrange
        $cart = new \App\ShoppingCart();

        // Act
        $cart->addItem(new \App\Product('PHP Book', 29.99));
        $cart->addItem(new \App\Product('Testing Guide', 19.99));

        // Assert
        $this->assertSame(49.98, $cart->getTotal());
    }
}
```


## Resources

- [PHPUnit Documentation - Writing Tests](https://docs.phpunit.de/en/12.0/writing-tests-for-phpunit.html) — PHPUnit official guide for writing and structuring tests

---

> 📘 *This lesson is part of the [PHP Testing & Quality Assurance](https://stanza.dev/courses/php-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*