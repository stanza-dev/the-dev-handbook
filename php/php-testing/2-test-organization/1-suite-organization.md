---
source_course: "php-testing"
source_lesson: "php-testing-suite-organization"
---

# Test Suite Organization

## Introduction
As a PHP project grows, so does its test suite. Without a clear directory structure and naming convention, tests become hard to find, slow to run, and impossible to maintain. This lesson covers the standard directory layout, phpunit.xml suite configuration, and naming conventions that professional PHP teams follow.

## Key Concepts
- **Test suite**: A named group of tests defined in `phpunit.xml`. Suites let you run subsets of your tests independently.
- **Unit tests**: Fast, isolated tests that verify a single class or function with no external dependencies.
- **Integration tests**: Tests that verify how multiple classes work together, often involving databases or file systems.
- **Feature tests**: End-to-end tests that exercise a complete user-facing workflow, such as an HTTP request through a controller.

## Real World Context
On a team of ten developers, each pushing code daily, you need the ability to run only unit tests in CI for fast feedback (under 30 seconds) and integration tests on merge. A well-organized test suite makes this trivial with a single `--testsuite Unit` flag.

## Deep Dive

### Standard Directory Structure

The most widely adopted layout mirrors the `src/` directory under a `tests/` root, separated by test type:

```
project/
├── src/
│   ├── Service/
│   │   └── OrderService.php
│   ├── Repository/
│   │   └── OrderRepository.php
│   └── Controller/
│       └── OrderController.php
├── tests/
│   ├── Unit/
│   │   └── Service/
│   │       └── OrderServiceTest.php
│   ├── Integration/
│   │   └── Repository/
│   │       └── OrderRepositoryTest.php
│   └── Feature/
│       └── OrderWorkflowTest.php
└── phpunit.xml
```

Each test class mirrors the namespace and location of the class it tests. `OrderServiceTest` lives under `tests/Unit/Service/` because `OrderService` lives under `src/Service/`.

### Configuring Test Suites in phpunit.xml

Define named suites so you can run them independently:

```xml
<phpunit xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:noNamespaceSchemaLocation="vendor/phpunit/phpunit/phpunit.xsd"
         bootstrap="vendor/autoload.php"
         colors="true"
         cacheDirectory=".phpunit.cache">
    <testsuites>
        <testsuite name="Unit">
            <directory>tests/Unit</directory>
        </testsuite>
        <testsuite name="Integration">
            <directory>tests/Integration</directory>
        </testsuite>
        <testsuite name="Feature">
            <directory>tests/Feature</directory>
        </testsuite>
    </testsuites>
    <source>
        <include>
            <directory>src</directory>
        </include>
    </source>
</phpunit>
```

Run a single suite with the `--testsuite` flag:

```bash
# Run only unit tests (fast, no external deps)
./vendor/bin/phpunit --testsuite Unit

# Run only integration tests
./vendor/bin/phpunit --testsuite Integration
```

This separation is essential for CI pipelines where unit tests run on every commit and integration tests run less frequently.

### Naming Conventions

Consistent naming makes tests self-documenting:

```php
<?php

// Class name: <ClassUnderTest>Test
class OrderServiceTest extends TestCase
{
    // Method name: describes the behaviour
    #[Test]
    public function createsOrderWithValidItems(): void { /* ... */ }

    #[Test]
    public function rejectsOrderWithEmptyCart(): void { /* ... */ }

    #[Test]
    public function appliesCouponDiscount(): void { /* ... */ }
}
```

Test class names end with `Test`. Method names describe what the class does, not what the test checks. Reading the method list should feel like reading a specification of the class.

### Autoloading Tests

Add a PSR-4 autoload section for tests in `composer.json`:

```json
{
    "autoload-dev": {
        "psr-4": {
            "Tests\\": "tests/"
        }
    }
}
```

Run `composer dump-autoload` after adding this. Now test classes in `tests/Unit/Service/OrderServiceTest.php` resolve to the namespace `Tests\Unit\Service\OrderServiceTest`.

## Common Pitfalls
1. **Putting all tests in a single directory** — Without separation, you cannot run fast unit tests independently from slow integration tests. Always split by test type.
2. **Naming test methods after implementation details** — `testCallsRepositoryFindMethod` couples the test to the implementation. If you refactor the repository call, the test name becomes misleading even if behaviour is unchanged.

## Best Practices
1. **Mirror the source tree** — If a class lives at `src/Payment/StripeGateway.php`, its test lives at `tests/Unit/Payment/StripeGatewayTest.php`. This makes it trivial to navigate between code and tests.
2. **Run unit tests in CI on every push** — Keep unit tests fast (under 1 second each) so the entire suite completes in seconds. Reserve integration and feature tests for merge pipelines.

## Summary
- Organize tests into `Unit/`, `Integration/`, and `Feature/` directories.
- Define named test suites in `phpunit.xml` and run them with `--testsuite`.
- Name test classes after the class they test, and methods after the behaviour they verify.
- Mirror the source directory structure under `tests/` for easy navigation.

## Code Examples

**A well-organized unit test following PSR-4 namespacing and mirror-directory conventions**

```php
<?php

declare(strict_types=1);

namespace Tests\Unit\Service;

use PHPUnit\Framework\Attributes\Test;
use PHPUnit\Framework\TestCase;
use App\Service\PasswordStrengthChecker;

class PasswordStrengthCheckerTest extends TestCase
{
    private PasswordStrengthChecker $checker;

    protected function setUp(): void
    {
        $this->checker = new PasswordStrengthChecker();
    }

    #[Test]
    public function rejectsPasswordShorterThanEightCharacters(): void
    {
        $result = $this->checker->evaluate('Ab1!');

        $this->assertFalse($result->isStrong);
        $this->assertContains('minimum 8 characters', $result->errors);
    }

    #[Test]
    public function acceptsStrongPassword(): void
    {
        $result = $this->checker->evaluate('S3cur3P@ssword!');

        $this->assertTrue($result->isStrong);
        $this->assertEmpty($result->errors);
    }
}
```


## Resources

- [PHPUnit 12 – Organizing Tests](https://docs.phpunit.de/en/12.0/organizing-tests.html) — Official PHPUnit guide on organizing test suites and directory structure
- [PSR-4 Autoloading Standard](https://www.php-fig.org/psr/psr-4/) — The PHP-FIG autoloading standard used for test namespace mapping

---

> 📘 *This lesson is part of the [PHP Testing & Quality Assurance](https://stanza.dev/courses/php-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*