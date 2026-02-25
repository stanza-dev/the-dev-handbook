---
source_course: "php-testing"
source_lesson: "php-testing-coverage"
---

# #[CoversClass] and Code Coverage

## Introduction
Code coverage tells you which lines, methods, and branches of your source code are actually executed by your tests. PHPUnit 12 uses the `#[CoversClass]` attribute to explicitly declare which class a test covers, ensuring your coverage reports are accurate rather than inflated by coincidental execution. This lesson explains coverage drivers, reports, and how to interpret the numbers.

## Key Concepts
- **Code coverage**: A metric showing the percentage of your source code executed during testing.
- **#[CoversClass]**: A PHP attribute that declares which production class a test class is responsible for covering.
- **Line coverage**: Percentage of executable source lines hit by tests.
- **Branch coverage**: Percentage of decision branches (if/else, switch) taken during tests.
- **Method coverage**: Percentage of methods called at least once during tests.
- **Coverage driver**: A PHP extension (Xdebug or PCOV) that instruments code to track execution.

## Real World Context
A codebase might show 90% line coverage but still have critical bugs in untested branches. Branch coverage reveals that your `if` block is tested but the `else` block is not. Teams that track both line and branch coverage catch more bugs and make more confident refactors.

## Deep Dive

### Setting Up a Coverage Driver

PHPUnit cannot collect coverage on its own. You need one of two PHP extensions:

```bash
# Option 1: PCOV (faster, recommended for CI)
pecl install pcov

# Option 2: Xdebug (also provides debugging and profiling)
pecl install xdebug
```

PCOV is purpose-built for coverage and adds minimal overhead. Xdebug is a full debugger that also supports coverage but runs slower. Most CI pipelines use PCOV for speed.

### The #[CoversClass] Attribute

Without `#[CoversClass]`, PHPUnit counts *all* code executed by a test toward coverage, including framework internals and helper classes. This inflates your numbers. The attribute scopes coverage to the declared class:

```php
<?php

declare(strict_types=1);

namespace Tests\Unit;

use PHPUnit\Framework\Attributes\CoversClass;
use PHPUnit\Framework\Attributes\Test;
use PHPUnit\Framework\TestCase;
use App\TaxCalculator;

#[CoversClass(TaxCalculator::class)]
class TaxCalculatorTest extends TestCase
{
    #[Test]
    public function calculatesStateTax(): void
    {
        $calculator = new TaxCalculator();

        $tax = $calculator->calculate(100.00, state: 'CA');

        $this->assertSame(7.25, $tax);
    }

    #[Test]
    public function returnsZeroForTaxFreeState(): void
    {
        $calculator = new TaxCalculator();

        $tax = $calculator->calculate(100.00, state: 'OR');

        $this->assertSame(0.0, $tax);
    }
}
```

Now only lines inside `TaxCalculator` are counted toward this test's coverage. If the test accidentally calls a `Logger` class, those lines do not inflate the report.

### Generating Coverage Reports

PHPUnit can output coverage in several formats:

```bash
# HTML report (human-readable, browse in a browser)
./vendor/bin/phpunit --coverage-html coverage-report/

# Text summary in the terminal
./vendor/bin/phpunit --coverage-text

# Clover XML (for CI tools like Codecov or SonarQube)
./vendor/bin/phpunit --coverage-clover coverage.xml
```

The HTML report is the most useful during development. It highlights covered lines in green and uncovered lines in red, and shows branch-level detail.

### Interpreting Coverage Metrics

Coverage reports show three main metrics:

```
Code Coverage Report:
  Classes:  80.00% (8/10)
  Methods:  75.00% (15/20)
  Lines:    82.35% (70/85)
```

- **Class coverage**: Percentage of classes that have at least one test covering them.
- **Method coverage**: Percentage of methods called at least once. If a class has 20 methods and 15 are called by tests, method coverage is 75%.
- **Line coverage**: Percentage of executable lines hit. This is the most commonly cited metric.

Branch coverage requires Xdebug and shows whether both sides of conditionals are tested.

### Enforcing Minimum Coverage

You can fail the build if coverage drops below a threshold:

```xml
<phpunit>
    <coverage>
        <report>
            <clover outputFile="coverage.xml"/>
        </report>
    </coverage>
    <source>
        <include>
            <directory>src</directory>
        </include>
    </source>
</phpunit>
```

Then in CI, use a tool like `phpunit-coverage-check` to enforce a minimum:

```bash
./vendor/bin/phpunit --coverage-clover coverage.xml
php phpunit-coverage-check coverage.xml 80
```

This fails the build if line coverage drops below 80%.

## Common Pitfalls
1. **Chasing 100% coverage** — 100% line coverage does not mean your code is bug-free. It means every line was executed, not that every edge case was tested. Focus on meaningful tests, not percentages.
2. **Forgetting #[CoversClass]** — Without it, coverage reports include lines from dependencies, making your numbers look better than they are. Always declare what each test class covers.

## Best Practices
1. **Use PCOV in CI, Xdebug locally** — PCOV is 2-5x faster for coverage collection. Use Xdebug when you need step-debugging during development, and PCOV for automated coverage in pipelines.
2. **Track coverage trends, not absolute numbers** — A coverage decrease on a pull request is a red flag. Use CI tools that comment on PRs with coverage deltas to catch regressions early.

## Summary
- Code coverage requires a driver: PCOV (fast) or Xdebug (full-featured).
- Use `#[CoversClass(ClassName::class)]` to scope coverage to the class under test.
- Generate HTML reports for local development, Clover XML for CI integration.
- Method coverage = methods called / total methods. Line coverage = lines hit / total executable lines.
- Aim for meaningful coverage, not an arbitrary 100% target.

## Code Examples

**A test class using #[CoversClass] to scope coverage reporting to DiscountEngine only, preventing inflated metrics**

```php
<?php

declare(strict_types=1);

namespace Tests\Unit;

use PHPUnit\Framework\Attributes\CoversClass;
use PHPUnit\Framework\Attributes\Test;
use PHPUnit\Framework\TestCase;
use App\DiscountEngine;

#[CoversClass(DiscountEngine::class)]
class DiscountEngineTest extends TestCase
{
    private DiscountEngine $engine;

    protected function setUp(): void
    {
        $this->engine = new DiscountEngine();
    }

    #[Test]
    public function appliesFlatDiscount(): void
    {
        $result = $this->engine->apply(
            subtotal: 200.00,
            discountType: 'flat',
            value: 25.00
        );

        $this->assertSame(175.00, $result);
    }

    #[Test]
    public function appliesPercentageDiscount(): void
    {
        $result = $this->engine->apply(
            subtotal: 200.00,
            discountType: 'percentage',
            value: 10
        );

        $this->assertSame(180.00, $result);
    }

    #[Test]
    public function neverReducesBelowZero(): void
    {
        $result = $this->engine->apply(
            subtotal: 10.00,
            discountType: 'flat',
            value: 50.00
        );

        $this->assertSame(0.00, $result);
    }
}
```


## Resources

- [PHPUnit 12 – Code Coverage](https://docs.phpunit.de/en/12.0/code-coverage.html) — Official PHPUnit documentation on code coverage analysis and reporting
- [PCOV – Code Coverage Driver](https://github.com/krakjoe/pcov) — A fast, lightweight PHP code coverage driver purpose-built for CI environments

---

> 📘 *This lesson is part of the [PHP Testing & Quality Assurance](https://stanza.dev/courses/php-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*