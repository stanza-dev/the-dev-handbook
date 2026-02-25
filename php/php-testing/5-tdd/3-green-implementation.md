---
source_course: "php-testing"
source_lesson: "php-testing-tdd-green-implementation"
---

# Making Tests Pass (Green Phase)

## Introduction
Once you have a failing test, the next step is to make it pass with the least amount of code possible. The Green phase is about satisfying the test contract, not about writing elegant or complete code. This lesson covers the strategies and discipline required to stay in the Green phase without over-engineering.

## Key Concepts
- **Green Phase**: The second step of TDD where you write the minimal production code to make the failing test pass.
- **Fake It Till You Make It**: A strategy where you return a hard-coded value to pass a test, then let additional tests force a real implementation.
- **Triangulation**: Writing multiple test cases that force the implementation to become general, rather than hard-coded for a single case.
- **Obvious Implementation**: When the correct code is simple and clear, you write it directly instead of faking.

## Real World Context
The discipline of writing minimal code prevents a common trap: spending hours building infrastructure you do not need. Many developers instinctively over-engineer solutions. The Green phase forces you to prove, through tests, that every line of code is necessary. This is the principle of YAGNI (You Aren't Gonna Need It) enforced by your test suite.

## Deep Dive

### Strategy 1: Fake It
Start by returning the exact value the test expects. This sounds silly, but it creates a green bar quickly and lets you move to the Refactor phase.

```php
<?php
// The test expects:
// $calculator->fibonacci(0) === 0

class Fibonacci
{
    public function calculate(int $n): int
    {
        return 0; // Fake it: hardcoded to pass the first test
    }
}
```

This passes the test for `fibonacci(0)`. Now add another test for `fibonacci(1)` to force the implementation forward.

### Strategy 2: Triangulate
Add more test cases that make the fake implementation fail, forcing you toward a real solution.

```php
<?php
class FibonacciTest extends TestCase
{
    public function testFibonacciOfZero(): void
    {
        $fib = new Fibonacci();
        $this->assertSame(0, $fib->calculate(0));
    }

    public function testFibonacciOfOne(): void
    {
        $fib = new Fibonacci();
        $this->assertSame(1, $fib->calculate(1));
    }

    public function testFibonacciOfSix(): void
    {
        $fib = new Fibonacci();
        $this->assertSame(8, $fib->calculate(6));
    }
}
```

With three tests, you can no longer fake it. The implementation must handle the general case.

```php
<?php
class Fibonacci
{
    public function calculate(int $n): int
    {
        if ($n <= 1) {
            return $n;
        }

        $previous = 0;
        $current = 1;

        for ($i = 2; $i <= $n; $i++) {
            $next = $previous + $current;
            $previous = $current;
            $current = $next;
        }

        return $current;
    }
}
```

All three tests now pass with a real implementation. The tests drove the design incrementally.

### Strategy 3: Obvious Implementation
When the solution is trivial, skip the faking step and write the real code immediately.

```php
<?php
// Test: $converter->celsiusToFahrenheit(0) === 32.0
// The formula is obvious, so write it directly

class TemperatureConverter
{
    public function celsiusToFahrenheit(float $celsius): float
    {
        return ($celsius * 9 / 5) + 32;
    }
}
```

Use Obvious Implementation only when you are confident the code is correct. If you are unsure, fall back to Fake It.

### Avoid Over-Engineering
The Green phase is not the place to add error handling, logging, caching, or other features your tests do not require. If no test demands it, do not build it.

```php
<?php
// Bad: over-engineering in the Green phase
class UserRepository
{
    private array $cache = [];
    private LoggerInterface $logger;

    public function findById(int $id): ?User
    {
        $this->logger->info("Finding user $id");
        if (isset($this->cache[$id])) {
            return $this->cache[$id];
        }
        // ... database query, caching, retry logic ...
    }
}

// Good: just enough to pass the test
class UserRepository
{
    public function findById(int $id): ?User
    {
        // Minimal implementation driven by the current test
        return $this->users[$id] ?? null;
    }
}
```

Caching, logging, and retries will be added when tests specifically require them.

## Common Pitfalls
1. **Writing more code than the test demands** — If your test only checks that `add(2, 3)` returns `5`, do not also handle negative numbers, floats, or overflow. Future tests will drive those requirements.
2. **Skipping straight to a complex implementation** — Jumping to the final solution bypasses the incremental design benefits of TDD. Start simple and let tests guide you toward complexity.

## Best Practices
1. **Use the simplest strategy that works** — If Fake It gets you to green, use it. If the solution is obvious, implement it directly. Match the strategy to your confidence level.
2. **Run tests after every small change** — The Green phase relies on fast feedback. Change one line, run the tests, confirm green. This tight loop catches mistakes instantly.

## Summary
- The Green phase means writing the absolute minimum code to make the failing test pass.
- "Fake It Till You Make It" returns hard-coded values, letting additional tests force a real implementation.
- Triangulation uses multiple test cases to push the implementation from specific to general.
- Obvious Implementation is appropriate only when the correct code is trivially clear.
- Never add features, error handling, or infrastructure that your tests do not require.

## Code Examples

**Triangulation strategy: four test cases force a general Fibonacci implementation, demonstrating how tests drive design incrementally**

```php
<?php
declare(strict_types=1);

use PHPUnit\Framework\TestCase;

// Triangulation: multiple tests drive a real implementation
class FibonacciTest extends TestCase
{
    private Fibonacci $fib;

    protected function setUp(): void
    {
        $this->fib = new Fibonacci();
    }

    public function testZeroReturnsZero(): void
    {
        $this->assertSame(0, $this->fib->calculate(0));
    }

    public function testOneReturnsOne(): void
    {
        $this->assertSame(1, $this->fib->calculate(1));
    }

    public function testSixReturnsEight(): void
    {
        $this->assertSame(8, $this->fib->calculate(6));
    }

    public function testTenReturnsFiftyFive(): void
    {
        $this->assertSame(55, $this->fib->calculate(10));
    }
}

// The implementation that satisfies all four tests
class Fibonacci
{
    public function calculate(int $n): int
    {
        if ($n <= 1) {
            return $n;
        }

        $previous = 0;
        $current = 1;
        for ($i = 2; $i <= $n; $i++) {
            $next = $previous + $current;
            $previous = $current;
            $current = $next;
        }

        return $current;
    }
}
```


## Resources

- [PHPUnit Documentation - Assertions](https://docs.phpunit.de/en/12.0/assertions.html) — PHPUnit assertion reference used to verify Green phase results
- [PHP Manual - Functions](https://www.php.net/manual/en/language.functions.php) — PHP functions reference, useful for understanding the implementation patterns tested in the Green phase

---

> 📘 *This lesson is part of the [PHP Testing & Quality Assurance](https://stanza.dev/courses/php-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*