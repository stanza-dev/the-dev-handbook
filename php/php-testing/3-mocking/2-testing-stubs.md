---
source_course: "php-testing"
source_lesson: "php-testing-stubs"
---

# Creating Stubs with PHPUnit

## Introduction
Stubs are the workhorse of test isolation. They let you replace a dependency with an object that returns pre-configured values, so you can control exactly what your system under test receives without touching real databases, APIs, or file systems.

## Key Concepts
- **createStub()**: PHPUnit method that creates a stub from a class or interface. Stub methods return `null` by default and will not fail the test if called unexpectedly.
- **willReturn()**: Configures a stubbed method to return a specific value.
- **willReturnMap()**: Returns different values based on the arguments passed to the stubbed method.
- **willReturnCallback()**: Delegates return-value computation to a callable, giving you full control over stub behavior.
- **willThrowException()**: Makes a stubbed method throw an exception instead of returning a value.

## Real World Context
Consider a `ShippingCalculator` that depends on a `WeightService` to look up package weights. In production, `WeightService` queries a warehouse database. During testing, you stub `WeightService` to return known weights so you can verify the shipping calculation logic without a database connection.

## Deep Dive
The simplest way to create a stub is with `createStub()`. Every method on the stub returns a type-appropriate default (`null`, `false`, `0`, or `[]`) until you configure it otherwise.

Here is a basic stub that returns a fixed value:

```php
<?php
declare(strict_types=1);

use PHPUnit\Framework\TestCase;

class ShippingCalculatorTest extends TestCase
{
    public function testCalculateShippingCost(): void
    {
        // Create a stub for the WeightService dependency
        $weightService = $this->createStub(WeightService::class);

        // Configure the stub to return a known weight
        $weightService->method('getWeight')
            ->willReturn(2.5);

        $calculator = new ShippingCalculator($weightService);
        $cost = $calculator->calculateCost('PKG-001');

        // 2.5 kg * $3.00/kg = $7.50
        $this->assertSame(7.50, $cost);
    }
}
```

The stub above always returns `2.5` regardless of the arguments. This is perfect when you only call the method once or don't care about argument variation.

When your code calls the same method with different arguments and expects different results, use `willReturnMap()`:

```php
<?php
public function testShippingForMultiplePackages(): void
{
    $weightService = $this->createStub(WeightService::class);

    // Map arguments to return values: [arg1, arg2, ..., returnValue]
    $weightService->method('getWeight')
        ->willReturnMap([
            ['PKG-001', 2.5],
            ['PKG-002', 1.0],
            ['PKG-003', 5.0],
        ]);

    $calculator = new ShippingCalculator($weightService);

    $this->assertSame(7.50, $calculator->calculateCost('PKG-001'));
    $this->assertSame(3.00, $calculator->calculateCost('PKG-002'));
    $this->assertSame(15.00, $calculator->calculateCost('PKG-003'));
}
```

Each inner array contains the expected arguments followed by the return value as the last element. If the arguments don't match any row, the stub returns `null`.

For more dynamic behavior, use `willReturnCallback()` with a closure:

```php
<?php
public function testDynamicStubBehavior(): void
{
    $taxService = $this->createStub(TaxService::class);

    // Use a callback for computed return values
    $taxService->method('calculateTax')
        ->willReturnCallback(function (float $amount, string $state): float {
            return match ($state) {
                'CA' => $amount * 0.0725,
                'TX' => $amount * 0.0625,
                'OR' => 0.0,  // No sales tax
                default => $amount * 0.05,
            };
        });

    $orderService = new OrderService($taxService);

    $this->assertEqualsWithDelta(7.25, $orderService->getTax(100.00, 'CA'), 0.01);
    $this->assertEqualsWithDelta(0.00, $orderService->getTax(100.00, 'OR'), 0.01);
}
```

The callback receives the same arguments that the stubbed method is called with, so you can compute realistic return values.

Finally, to test error paths, make the stub throw an exception:

```php
<?php
public function testHandlesServiceFailure(): void
{
    $weightService = $this->createStub(WeightService::class);

    $weightService->method('getWeight')
        ->willThrowException(new \RuntimeException('Service unavailable'));

    $calculator = new ShippingCalculator($weightService);

    $this->expectException(ShippingException::class);
    $this->expectExceptionMessage('Could not calculate shipping');

    $calculator->calculateCost('PKG-001');
}
```

This lets you verify that your code handles failures gracefully without needing to cause a real service outage.

## Common Pitfalls
1. **Forgetting the return value in willReturnMap()** — The last element in each inner array is the return value, not an argument. Writing `['PKG-001']` without a return value causes the stub to return `null` silently, leading to confusing test failures.
2. **Using createMock() when createStub() suffices** — If you don't need to verify that a method was called, use `createStub()`. Mocks carry expectations that can cause unexpected test failures when the implementation changes slightly.

## Best Practices
1. **Start with the simplest stub** — Use `willReturn()` first. Only reach for `willReturnMap()` or `willReturnCallback()` when the test genuinely requires argument-dependent responses.
2. **Stub at the interface boundary** — Stub interfaces rather than concrete classes. This ensures your tests depend on contracts, not implementations, and makes refactoring safer.

## Summary
- `createStub()` creates a test double whose methods return defaults and never fail the test.
- `willReturn()` sets a fixed return value; `willReturnMap()` maps arguments to values.
- `willReturnCallback()` provides full control via a closure when you need computed results.
- `willThrowException()` lets you test error-handling paths.
- Always prefer the simplest stubbing approach that satisfies your test's needs.

## Code Examples

**Stubbing an interface with willReturnMap() to return different values based on currency conversion arguments**

```php
<?php
declare(strict_types=1);

use PHPUnit\Framework\TestCase;

interface CurrencyConverterInterface
{
    public function convert(float $amount, string $from, string $to): float;
    public function getRate(string $from, string $to): float;
}

class InvoiceServiceTest extends TestCase
{
    public function testConvertInvoiceToTargetCurrency(): void
    {
        // Stub the currency converter with a return map
        $converter = $this->createStub(CurrencyConverterInterface::class);
        $converter->method('convert')
            ->willReturnMap([
                [100.00, 'USD', 'EUR', 92.50],
                [100.00, 'USD', 'GBP', 79.30],
                [250.00, 'USD', 'EUR', 231.25],
            ]);

        $invoiceService = new InvoiceService($converter);

        $euroInvoice = $invoiceService->convertTotal(100.00, 'USD', 'EUR');
        $this->assertSame(92.50, $euroInvoice);

        $poundInvoice = $invoiceService->convertTotal(100.00, 'USD', 'GBP');
        $this->assertSame(79.30, $poundInvoice);
    }
}
?>
```


## Resources

- [PHPUnit Stubs Documentation](https://docs.phpunit.de/en/12.0/test-doubles.html#stubs) — Official PHPUnit 12 documentation on creating and configuring stubs

---

> 📘 *This lesson is part of the [PHP Testing & Quality Assurance](https://stanza.dev/courses/php-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*