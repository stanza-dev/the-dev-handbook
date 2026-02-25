---
source_course: "php-testing"
source_lesson: "php-testing-first-test"
---

# Writing Your First Test

## Introduction
PHPUnit 12 is the industry-standard testing framework for PHP. In this lesson you will install PHPUnit, configure it, and write your very first test using the modern `#[Test]` attribute syntax. By the end you will have a working test suite you can run from the command line.

## Key Concepts
- **PHPUnit**: The de facto unit-testing framework for PHP, providing a runner, assertions, and test lifecycle hooks.
- **TestCase**: The base class every test class extends. It gives you access to assertions and lifecycle methods.
- **#[Test] attribute**: A PHP 8 attribute that marks a method as a test case, replacing the older `@test` docblock annotation.
- **Arrange-Act-Assert (AAA)**: A three-step pattern for structuring every test: set up state, perform the action, verify the outcome.

## Real World Context
Every professional PHP project ships with an automated test suite. Without tests, a simple refactor can silently break payment processing, user authentication, or data exports. PHPUnit gives you a safety net that catches regressions before they reach production.

## Deep Dive

### Installing PHPUnit 12

PHPUnit 12 requires PHP 8.3 or higher. Install it as a development dependency with Composer:

```bash
composer require --dev phpunit/phpunit:^12.0
```

After installation, the `phpunit` binary is available at `./vendor/bin/phpunit`.

### Configuring phpunit.xml

PHPUnit reads its configuration from a `phpunit.xml` file in your project root. Here is a minimal configuration for PHPUnit 12:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<phpunit xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:noNamespaceSchemaLocation="vendor/phpunit/phpunit/phpunit.xsd"
         bootstrap="vendor/autoload.php"
         colors="true"
         cacheDirectory=".phpunit.cache">
    <testsuites>
        <testsuite name="Unit">
            <directory>tests/Unit</directory>
        </testsuite>
    </testsuites>
    <source>
        <include>
            <directory>src</directory>
        </include>
    </source>
</phpunit>
```

The `bootstrap` attribute loads Composer's autoloader so your classes are available in tests. The `cacheDirectory` attribute tells PHPUnit 12 where to store its result cache for faster re-runs.

### Writing Your First Test

Every test class extends `PHPUnit\Framework\TestCase`. Mark individual methods as tests with the `#[Test]` attribute:

```php
<?php

declare(strict_types=1);

namespace Tests\Unit;

use PHPUnit\Framework\Attributes\Test;
use PHPUnit\Framework\TestCase;
use App\Calculator;

class CalculatorTest extends TestCase
{
    #[Test]
    public function addsTwoPositiveNumbers(): void
    {
        // Arrange
        $calculator = new Calculator();

        // Act
        $result = $calculator->add(3, 7);

        // Assert
        $this->assertSame(10, $result);
    }

    #[Test]
    public function subtractsSmallFromLarge(): void
    {
        $calculator = new Calculator();

        $result = $calculator->subtract(10, 4);

        $this->assertSame(6, $result);
    }
}
```

Notice the AAA pattern in each method: we arrange the objects, act by calling a method, and assert the result matches our expectation. The `#[Test]` attribute from `PHPUnit\Framework\Attributes\Test` replaces both the `test` method-name prefix and the legacy `@test` docblock annotation.

### Running Tests

Execute your entire suite or target a specific file or method:

```bash
# Run all tests
./vendor/bin/phpunit

# Run a specific test file
./vendor/bin/phpunit tests/Unit/CalculatorTest.php

# Run a single test method by name
./vendor/bin/phpunit --filter addsTwoPositiveNumbers
```

PHPUnit outputs a dot for each passing test and an `F` for each failure, followed by a summary.

## Common Pitfalls
1. **Using `@test` annotations instead of `#[Test]` attributes** — PHPUnit 12 favours PHP attributes over docblock annotations. Attributes are checked by the PHP engine itself, giving you IDE autocompletion and compile-time safety that annotations lack.
2. **Forgetting the return type `void`** — Test methods must declare `void` as their return type. Omitting it may trigger deprecation warnings in PHPUnit 12.

## Best Practices
1. **One assertion concept per test** — Each test should verify a single behaviour. If you need to check two independent outcomes, write two test methods. This makes failure messages precise.
2. **Name tests after the behaviour, not the method** — `addsTwoPositiveNumbers` is better than `testAdd`. Descriptive names serve as living documentation of what the class does.

## Summary
- Install PHPUnit 12 via Composer and configure it with `phpunit.xml`.
- Extend `TestCase` and mark methods with the `#[Test]` attribute.
- Follow the Arrange-Act-Assert pattern to keep tests readable.
- Run tests with `./vendor/bin/phpunit` and use `--filter` to target specific methods.

## Code Examples

**A realistic PHPUnit 12 test class using #[Test] attributes, setUp(), and assertSame for an order calculator**

```php
<?php

declare(strict_types=1);

namespace Tests\Unit;

use PHPUnit\Framework\Attributes\Test;
use PHPUnit\Framework\TestCase;
use App\OrderCalculator;

class OrderCalculatorTest extends TestCase
{
    private OrderCalculator $orderCalculator;

    protected function setUp(): void
    {
        $this->orderCalculator = new OrderCalculator();
    }

    #[Test]
    public function calculatesSubtotalForMultipleItems(): void
    {
        $items = [
            ['price' => 29.99, 'quantity' => 2],
            ['price' => 9.99,  'quantity' => 1],
        ];

        $subtotal = $this->orderCalculator->subtotal($items);

        // 29.99 * 2 + 9.99 * 1 = 69.97
        $this->assertSame(69.97, $subtotal);
    }

    #[Test]
    public function returnsZeroForEmptyCart(): void
    {
        $subtotal = $this->orderCalculator->subtotal([]);

        $this->assertSame(0.0, $subtotal);
    }
}
```


## Resources

- [PHPUnit 12 – Writing Tests](https://docs.phpunit.de/en/12.0/writing-tests-for-phpunit.html) — Official PHPUnit 12 guide on writing and running tests
- [PHP Attributes – PHP Manual](https://www.php.net/manual/en/language.attributes.overview.php) — PHP manual page explaining the attributes language feature used by PHPUnit 12

---

> 📘 *This lesson is part of the [PHP Testing & Quality Assurance](https://stanza.dev/courses/php-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*