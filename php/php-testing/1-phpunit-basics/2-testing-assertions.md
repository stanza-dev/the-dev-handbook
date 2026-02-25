---
source_course: "php-testing"
source_lesson: "php-testing-assertions"
---

# Understanding Assertions

## Introduction
Assertions are the verdict of every test. They compare an expected value against an actual value and either pass silently or fail with a descriptive message. PHPUnit ships with dozens of built-in assertion methods, and knowing which one to reach for makes your tests both clearer and more maintainable.

## Key Concepts
- **assertEquals**: Compares two values with loose (`==`) comparison, allowing type juggling.
- **assertSame**: Compares two values with strict (`===`) comparison, requiring both value and type to match.
- **assertTrue / assertFalse**: Verify a value is exactly `true` or `false`.
- **assertNull**: Confirms a value is `null`.
- **assertCount**: Checks the number of elements in a countable (array, `Countable` object).
- **assertInstanceOf**: Verifies that an object is an instance of a given class or interface.

## Real World Context
Choosing the wrong assertion leads to tests that pass when they should fail. For example, `assertEquals(0, false)` passes because `0 == false` in PHP. Using `assertSame(0, false)` correctly catches the type mismatch. In payment or security code, this distinction can mean the difference between a passing test suite and a production bug.

## Deep Dive

### assertEquals vs assertSame

The most important distinction in PHPUnit assertions is loose vs strict comparison. Consider the following test:

```php
<?php

$this->assertEquals(0, '');    // PASSES — 0 == '' is true in PHP
$this->assertSame(0, '');      // FAILS  — 0 !== ''

$this->assertEquals(1, true);  // PASSES — 1 == true
$this->assertSame(1, true);    // FAILS  — int !== bool
```

Because PHP's type juggling can produce surprising equalities, `assertSame` is almost always the safer choice when you know the exact type of the return value.

### Boolean and Null Assertions

Use dedicated methods instead of generic equality checks. They produce clearer failure messages:

```php
<?php

$userService = new UserService();
$isActive = $userService->isActive($userId);

// Preferred — failure says "Failed asserting that false is true"
$this->assertTrue($isActive);

// Avoid — failure says "Failed asserting that false matches expected true"
$this->assertEquals(true, $isActive);

// Null check
$deletedUser = $userService->findById($unknownId);
$this->assertNull($deletedUser);
```

Dedicated assertions also make the test's intent obvious at a glance.

### Collection Assertions

`assertCount` is clearer than manually calling `count()` inside `assertSame`:

```php
<?php

$orderRepository = new OrderRepository();
$recentOrders = $orderRepository->findRecentByUser($userId);

// Preferred
$this->assertCount(3, $recentOrders);

// Less readable alternative
$this->assertSame(3, count($recentOrders));
```

The first form produces a better failure message: "Failed asserting that actual size 5 matches expected size 3."

### Type Assertions

`assertInstanceOf` verifies that a factory or container returns the correct type:

```php
<?php

$logger = LoggerFactory::create('file');
$this->assertInstanceOf(FileLogger::class, $logger);

$cache = CacheFactory::create('redis');
$this->assertInstanceOf(CacheInterface::class, $cache);
```

This is especially useful when testing factories, service containers, or any code that returns interfaces.

## Common Pitfalls
1. **Using assertEquals when you need assertSame** — Loose comparison hides type bugs. Default to `assertSame` for scalars and switch to `assertEquals` only when you intentionally want loose comparison (e.g., comparing objects by value).
2. **Asserting too many things in one test** — If a test has ten assertions, the first failure masks the rest. Split the test or use a data provider to test each case independently.

## Best Practices
1. **Put expected value first** — PHPUnit's assertion signature is `assertSame($expected, $actual)`. Reversing the arguments produces confusing failure messages like "expected 5 but got 10" when you meant the opposite.
2. **Use the most specific assertion available** — Prefer `assertCount` over `assertSame(3, count(...))`, `assertStringContainsString` over a manual `strpos` check. Specific assertions yield better failure diagnostics.

## Summary
- `assertSame` uses strict `===` comparison; `assertEquals` uses loose `==`.
- Prefer `assertSame` for scalar values to avoid PHP type-juggling surprises.
- Use dedicated assertions (`assertTrue`, `assertNull`, `assertCount`, `assertInstanceOf`) for clearer intent and better failure messages.
- Always place the expected value as the first argument.

## Code Examples

**Demonstrates assertInstanceOf, assertSame, assertCount, assertFalse, and assertNull in a single realistic test**

```php
<?php

declare(strict_types=1);

namespace Tests\Unit;

use PHPUnit\Framework\Attributes\Test;
use PHPUnit\Framework\TestCase;
use App\InvoiceGenerator;
use App\Invoice;

class InvoiceGeneratorTest extends TestCase
{
    #[Test]
    public function generatesInvoiceWithCorrectTotal(): void
    {
        $generator = new InvoiceGenerator();

        $invoice = $generator->create(
            customerId: 42,
            lineItems: [
                ['description' => 'Widget', 'amount' => 19.99],
                ['description' => 'Gadget', 'amount' => 49.99],
            ]
        );

        $this->assertInstanceOf(Invoice::class, $invoice);
        $this->assertSame(42, $invoice->customerId);
        $this->assertSame(69.98, $invoice->total);
        $this->assertCount(2, $invoice->lineItems);
        $this->assertFalse($invoice->isPaid());
        $this->assertNull($invoice->paidAt);
    }
}
```


## Resources

- [PHPUnit 12 – Assertions](https://docs.phpunit.de/en/12.0/assertions.html) — Complete reference of all PHPUnit assertion methods
- [PHP Type Comparison Tables](https://www.php.net/manual/en/types.comparisons.php) — Official PHP comparison tables showing == vs === behaviour

---

> 📘 *This lesson is part of the [PHP Testing & Quality Assurance](https://stanza.dev/courses/php-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*