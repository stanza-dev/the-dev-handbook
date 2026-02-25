---
source_course: "php-testing"
source_lesson: "php-testing-mutation-testing"
---

# Mutation Testing with Infection

## Introduction
Code coverage tells you which lines your tests execute, but not whether your tests would catch a bug on those lines. Mutation testing solves this by deliberately introducing bugs (mutations) into your code and checking if your tests detect them. If a test suite has 100% coverage but passes when a `>` is changed to `>=`, those tests are weak.

## Key Concepts
- **Mutation Testing**: A technique that modifies (mutates) your source code and runs your test suite to see if the tests catch the change.
- **Mutant**: A version of your code with one small change (e.g., replacing `+` with `-`, or `true` with `false`).
- **Killed Mutant**: A mutant that your tests detect (at least one test fails). This is the goal.
- **Escaped Mutant**: A mutant that your tests do not detect (all tests still pass). This reveals a gap in your test suite.
- **MSI (Mutation Score Indicator)**: The percentage of mutants killed. An MSI of 80% means your tests caught 80% of introduced bugs.
- **Infection**: The most popular mutation testing framework for PHP.

## Real World Context
A team had 95% code coverage but kept shipping bugs. The problem was that their tests executed the code without meaningfully asserting the results. Mutation testing exposed this: Infection showed an MSI of only 40%, meaning 60% of deliberate bugs went undetected. After strengthening their assertions, their MSI rose to 85% and production bugs dropped significantly.

## Deep Dive

### Installation
Install Infection as a dev dependency.

```bash
composer require --dev infection/infection
```

Run Infection against your test suite.

```bash
./vendor/bin/infection --min-msi=70 --min-covered-msi=80 --threads=4
```

The `--min-msi=70` flag makes Infection fail if the overall MSI drops below 70%. The `--threads=4` flag runs mutations in parallel for speed.

### How Mutations Work
Infection applies small changes called mutators. Each mutator targets a specific code pattern.

Consider this function and its test.

```php
<?php
class PriceCalculator
{
    public function applyDiscount(float $price, float $discount): float
    {
        if ($discount > 100) {
            $discount = 100;
        }

        return $price * (1 - $discount / 100);
    }
}
```

Infection creates mutants like these.

```php
<?php
// Mutant 1: Change > to >=
if ($discount >= 100) { ... }

// Mutant 2: Change > to <
if ($discount < 100) { ... }

// Mutant 3: Change - to +
return $price * (1 + $discount / 100);

// Mutant 4: Change * to /
return $price / (1 - $discount / 100);

// Mutant 5: Replace 100 with 0
if ($discount > 0) { ... }
```

Each mutant is a separate test run. If your tests catch the change (at least one test fails), the mutant is killed. If all tests still pass, the mutant escaped.

### Reading Infection Output
After running, Infection produces a summary.

```text
125 mutations were generated:
     105 mutants were killed
      10 mutants were not covered by tests
       7 mutants escaped
       3 mutants timed out

Metrics:
    Mutation Score Indicator (MSI): 84%
    Mutation Code Coverage: 92%
    Covered Code MSI: 91%
```

The important metric is MSI. An MSI of 84% means 84% of all mutations were detected. The Covered Code MSI (91%) only considers mutations in code that has test coverage, which is a better indicator of test quality for partially-tested codebases.

### Strengthening Tests Based on Infection Results
Infection logs escaped mutants so you can see exactly which mutations your tests missed.

```php
<?php
// Weak test: only checks the happy path
public function testApplyDiscount(): void
{
    $calculator = new PriceCalculator();
    $result = $calculator->applyDiscount(100.0, 20.0);
    $this->assertSame(80.0, $result);
}

// Stronger test suite: covers edge cases that kill more mutants
public function testApplyDiscountReducesPrice(): void
{
    $calculator = new PriceCalculator();
    $this->assertSame(80.0, $calculator->applyDiscount(100.0, 20.0));
}

public function testDiscountOverHundredIsCappedAtHundred(): void
{
    $calculator = new PriceCalculator();
    // This kills the > vs >= mutant
    $this->assertSame(0.0, $calculator->applyDiscount(100.0, 150.0));
}

public function testZeroDiscountReturnsOriginalPrice(): void
{
    $calculator = new PriceCalculator();
    $this->assertSame(100.0, $calculator->applyDiscount(100.0, 0.0));
}

public function testExactlyHundredPercentDiscountGivesFreeItem(): void
{
    $calculator = new PriceCalculator();
    // This kills the > vs >= boundary mutant
    $this->assertSame(0.0, $calculator->applyDiscount(50.0, 100.0));
}
```

The stronger test suite tests the boundaries (0%, 100%, over 100%) which kills mutations that change comparison operators.

### Configuration
Configure Infection with an `infection.json5` file.

```json
{
    "source": {
        "directories": ["src"]
    },
    "logs": {
        "text": "infection.log",
        "summary": "infection-summary.log"
    },
    "mutators": {
        "@default": true
    },
    "minMsi": 70,
    "minCoveredMsi": 80
}
```

The `@default` mutator set includes all standard mutations. You can disable specific mutators if they produce too many false positives for your codebase.

## Common Pitfalls
1. **Confusing code coverage with test quality** — 100% code coverage does not mean your tests are good. Coverage says "this line was executed" but not "this line was verified." Mutation testing measures whether your assertions actually catch bugs.
2. **Running Infection without sufficient coverage first** — Mutation testing is slow because it runs your entire test suite for each mutant. If coverage is low, most mutants will be in uncovered code and the results will be noise. Aim for at least 70% coverage before adding mutation testing.

## Best Practices
1. **Focus on Covered Code MSI** — The Covered Code MSI is more actionable than overall MSI because it only measures test quality for code you have already tested. Work on killing escaped mutants in covered code first.
2. **Use Infection in CI with minimum thresholds** — Set `--min-msi=70` in CI to prevent merging code with weak tests. Increase the threshold gradually as your test suite improves.

## Summary
- Mutation testing introduces deliberate bugs to verify your tests actually catch them.
- Infection PHP is the standard mutation testing tool, generating mutants and measuring which ones your tests kill.
- MSI (Mutation Score Indicator) is the key metric: aim for at least 70-80%.
- Escaped mutants reveal weak tests that execute code without meaningful assertions.
- Code coverage measures execution; mutation testing measures verification quality.

## Code Examples

**A mutation-resilient test suite for a price calculator, with comments explaining which mutants each test kills**

```php
<?php
declare(strict_types=1);

use PHPUnit\Framework\TestCase;

// The class under test
class PriceCalculator
{
    public function applyDiscount(float $price, float $discount): float
    {
        if ($discount > 100) {
            $discount = 100;
        }
        return $price * (1 - $discount / 100);
    }
}

// Mutation-resilient test suite
class PriceCalculatorTest extends TestCase
{
    private PriceCalculator $calculator;

    protected function setUp(): void
    {
        $this->calculator = new PriceCalculator();
    }

    public function testStandardDiscountReducesPrice(): void
    {
        // Kills arithmetic mutants (+ instead of -, * instead of /)
        $this->assertSame(80.0, $this->calculator->applyDiscount(100.0, 20.0));
    }

    public function testZeroDiscountReturnsFullPrice(): void
    {
        // Kills boundary mutants on the discount value
        $this->assertSame(100.0, $this->calculator->applyDiscount(100.0, 0.0));
    }

    public function testHundredPercentDiscountReturnsFree(): void
    {
        // Kills > vs >= mutant at the boundary
        $this->assertSame(0.0, $this->calculator->applyDiscount(50.0, 100.0));
    }

    public function testOverHundredPercentIsCapped(): void
    {
        // Kills mutants that remove the cap logic entirely
        $this->assertSame(0.0, $this->calculator->applyDiscount(75.0, 200.0));
    }
}
```


## Resources

- [Infection PHP Documentation](https://infection.github.io/guide/) — Official Infection documentation covering installation, configuration, and mutator reference
- [PHPUnit Code Coverage](https://docs.phpunit.de/en/12.0/code-coverage.html) — PHPUnit's code coverage guide, the foundation that mutation testing builds upon

---

> 📘 *This lesson is part of the [PHP Testing & Quality Assurance](https://stanza.dev/courses/php-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*