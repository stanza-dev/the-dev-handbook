---
source_course: "php-testing"
source_lesson: "php-testing-test-output"
---

# Understanding Test Output

## Introduction
When you run your test suite, PHPUnit prints a stream of characters to the terminal — dots, letters, and sometimes colourful stack traces. Understanding this output is essential for diagnosing failures quickly. This lesson teaches you to read every symbol PHPUnit produces, interpret failure messages, and use verbosity flags to get more detail when you need it.

## Key Concepts
- **`.` (dot)**: The test passed. This is the most common symbol and the one you want to see.
- **`F` (failure)**: An assertion failed. The expected value did not match the actual value.
- **`E` (error)**: The test threw an unexpected exception that was not caught by an `expectException()` call.
- **`S` (skipped)**: The test was skipped, usually because a required extension is not loaded or a condition was not met.
- **`R` (risky)**: The test is risky — it produced no assertions, or it manipulated global state without declaring it.
- **`I` (incomplete)**: The test was explicitly marked incomplete with `$this->markTestIncomplete()`.

## Real World Context
A CI pipeline that shows 847 dots and one `F` is easy to triage — you scroll to the failure summary at the bottom. But if you do not understand what `E` vs `F` means, you might waste time looking for a wrong assertion when the real problem is an unhandled exception. Reading output fluently is a skill that saves minutes on every test run.

## Deep Dive

### The Progress Line

When PHPUnit runs, it prints one character per test on a progress line:

```
.....F...E..S..........R.....                           28 / 28 (100%)
```

The counter on the right (`28 / 28`) shows how many tests have executed out of the total. Each character corresponds to one test method in the order it was executed. A fully green run looks like a row of dots.

### Failure Output (F)

When a test fails, PHPUnit prints the full details after the progress line:

```
1) Tests\Unit\CalculatorTest::addsTwoPositiveNumbers
Failed asserting that 11 is identical to 10.

/app/tests/Unit/CalculatorTest.php:25
```

The output tells you exactly which test failed, what the assertion expected versus what it received, and the file and line number. The format is always: the assertion description first, then the stack trace pointing to the failing line.

Here is how a comparison failure looks for more complex values:

```
2) Tests\Unit\OrderServiceTest::calculatesTotal
Failed asserting that two arrays are identical.
--- Expected
+++ Actual
@@ @@
 Array (
-    'total' => 99.99
+    'total' => 100.00
     'currency' => 'USD'
 )

/app/tests/Unit/OrderServiceTest.php:42
```

The diff format uses `-` for expected and `+` for actual, identical to a git diff. Lines without a prefix are unchanged.

### Error Output (E)

An error means the test threw an exception that was not expected:

```
3) Tests\Unit\PaymentTest::processesRefund
App\Exception\GatewayTimeoutException: Connection timed out after 30s

/app/src/Payment/Gateway.php:88
/app/src/Payment/PaymentService.php:45
/app/tests/Unit/PaymentTest.php:31
```

Unlike a failure (where the assertion itself did not match), an error means your code crashed. The stack trace shows the full call chain from the test method down to the line that threw.

### The Summary Footer

After all tests run, PHPUnit prints a summary:

```
Time: 00:02.347, Memory: 24.00 MB

Tests: 28, Assertions: 64, Failures: 1, Errors: 1, Skipped: 1, Risky: 1.
```

The footer gives you the total execution time, peak memory usage, and a count of every outcome category. If everything passes, you see:

```
OK (28 tests, 64 assertions)
```

This single line confirms that every test and every assertion passed.

### Verbose and Debug Flags

When the default output is not enough, PHPUnit provides flags for more detail:

```bash
# Show the name of every test as it runs
./vendor/bin/phpunit --display-deprecations --display-notices --display-warnings

# Show detailed information about test events
./vendor/bin/phpunit --debug
```

The `--debug` flag prints each test's fully qualified name as it starts and finishes, which is useful for finding tests that hang or produce unexpected output. In PHPUnit 12 the `--debug` flag outputs event-based trace information showing exactly when each test begins and ends.

### Understanding Risky Tests

A test is marked risky (`R`) when PHPUnit's strictness checks detect a problem. The most common causes are:

```php
<?php

#[Test]
public function loadsConfiguration(): void
{
    $config = new Config();
    $config->load('app.php');
    // No assertion! PHPUnit marks this as risky.
}
```

The test above has no assertion, so PHPUnit cannot know whether it actually verified anything. Fix it by adding an assertion that confirms the expected outcome:

```php
<?php

#[Test]
public function loadsConfiguration(): void
{
    $config = new Config();
    $config->load('app.php');

    $this->assertSame('production', $config->get('app.env'));
}
```

With the assertion in place, the test is no longer risky.

## Common Pitfalls
1. **Confusing `F` (failure) with `E` (error)** — A failure means an assertion did not match. An error means an exception was thrown. The fix for each is different: failures require correcting your logic or your expectation, while errors require handling or expecting the exception.
2. **Ignoring risky tests** — A test with no assertions is a false safety net. It will always pass regardless of what the code does. Enable `failOnRisky="true"` in `phpunit.xml` to treat risky tests as failures.

## Best Practices
1. **Read the diff carefully** — PHPUnit's failure diffs use `-` for expected and `+` for actual. Train yourself to read these like git diffs so you can spot the discrepancy in seconds.
2. **Enable strict mode in phpunit.xml** — Set `failOnWarning="true"`, `failOnRisky="true"`, and `beStrictAboutTestsThatDoNotTestAnything="true"` to catch risky and empty tests before they accumulate.

## Summary
- A dot (`.`) means pass, `F` means assertion failure, `E` means unexpected exception, `S` means skipped, and `R` means risky.
- Failure output shows the expected vs actual values and the exact file and line number.
- Error output shows the full exception stack trace from the throwing line to the test method.
- Use `--debug` for event-level tracing when diagnosing hangs or output issues.
- Enable strict mode in `phpunit.xml` so risky tests with no assertions are treated as failures.

## Code Examples

**Three tests demonstrating the output symbols — a passing test produces '.', a wrong value produces 'F' with a diff, and a missing exception produces 'E'**

```php
<?php

declare(strict_types=1);

namespace Tests\Unit;

use PHPUnit\Framework\Attributes\Test;
use PHPUnit\Framework\TestCase;
use App\TemperatureConverter;

class TemperatureConverterTest extends TestCase
{
    private TemperatureConverter $converter;

    protected function setUp(): void
    {
        $this->converter = new TemperatureConverter();
    }

    #[Test]
    public function convertsCelsiusToFahrenheit(): void
    {
        // This assertion will produce a clear diff if it fails:
        // "Failed asserting that 211.0 is identical to 212.0."
        $this->assertSame(212.0, $this->converter->toFahrenheit(100.0));
    }

    #[Test]
    public function throwsOnAbsoluteZeroViolation(): void
    {
        // If this exception is NOT thrown, the output shows:
        // "Failed asserting that exception of type
        //  InvalidArgumentException is thrown."
        $this->expectException(\InvalidArgumentException::class);

        $this->converter->toFahrenheit(-274.0);
    }

    #[Test]
    public function convertsFreezingPoint(): void
    {
        // A passing test produces a single dot: .
        $this->assertSame(32.0, $this->converter->toFahrenheit(0.0));
    }
}
```


## Resources

- [PHPUnit 12 – The Command-Line Test Runner](https://docs.phpunit.de/en/12.0/textui.html) — Official documentation for PHPUnit's command-line output, flags, and result interpretation
- [PHPUnit 12 – Risky Tests](https://docs.phpunit.de/en/12.0/risky-tests.html) — Official guide on what makes a test risky and how to enable strict mode

---

> 📘 *This lesson is part of the [PHP Testing & Quality Assurance](https://stanza.dev/courses/php-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*