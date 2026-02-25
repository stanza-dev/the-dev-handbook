---
source_course: "php-testing"
source_lesson: "php-testing-groups"
---

# Grouping and Filtering Tests

## Introduction
As your test suite grows to hundreds or thousands of tests, you need fine-grained control over which tests run and when. PHPUnit's `#[Group]` attribute and command-line filters let you tag tests by concern and run precisely the subset you need, whether that is all database tests, all slow tests, or a single class.

## Key Concepts
- **#[Group('name')]**: A PHP attribute that assigns a test method or class to a named group.
- **--group flag**: Runs only tests belonging to the specified group.
- **--exclude-group flag**: Runs all tests *except* those in the specified group.
- **--filter flag**: Runs tests whose fully qualified name matches a regex pattern.

## Real World Context
In continuous integration, you want fast feedback. Tagging slow integration tests with `#[Group('slow')]` lets your CI pipeline run `--exclude-group slow` on every push for a 10-second feedback loop, while a nightly job runs the full suite including slow tests.

## Deep Dive

### Assigning Groups with Attributes

Apply `#[Group]` at the class level (all methods inherit it) or at the method level:

```php
<?php

declare(strict_types=1);

namespace Tests\Integration;

use PHPUnit\Framework\Attributes\Group;
use PHPUnit\Framework\Attributes\Test;
use PHPUnit\Framework\TestCase;
use App\Repository\ProductRepository;

#[Group('database')]
class ProductRepositoryTest extends TestCase
{
    #[Test]
    public function findsProductById(): void
    {
        $repository = new ProductRepository($this->pdo);

        $product = $repository->findById(1);

        $this->assertSame('Wireless Mouse', $product->name);
    }

    #[Test]
    #[Group('slow')]
    public function reindexesEntireCatalog(): void
    {
        $repository = new ProductRepository($this->pdo);

        $count = $repository->reindexAll();

        $this->assertGreaterThan(0, $count);
    }
}
```

The class-level `#[Group('database')]` applies to every method. The second test also belongs to the `slow` group. A test can belong to multiple groups.

### Running Groups from the Command Line

Use `--group` and `--exclude-group` to control which tests execute:

```bash
# Run only database tests
./vendor/bin/phpunit --group database

# Run everything except slow tests
./vendor/bin/phpunit --exclude-group slow

# Run tests in multiple groups (OR logic)
./vendor/bin/phpunit --group database --group api
```

These flags combine with `--testsuite`, so you can run `--testsuite Integration --group database` for maximum precision.

### Filtering by Name

The `--filter` flag accepts a regex pattern matched against the fully qualified test name:

```bash
# Run all tests in ProductRepositoryTest
./vendor/bin/phpunit --filter ProductRepositoryTest

# Run tests whose name contains 'reindex'
./vendor/bin/phpunit --filter reindex

# Run a specific method in a specific class
./vendor/bin/phpunit --filter 'ProductRepositoryTest::findsProductById'
```

This is especially useful during development when you are working on one feature and want to run only its related tests.

### Configuring Default Groups in phpunit.xml

You can exclude groups by default in your configuration, so they only run when explicitly requested:

```xml
<phpunit>
    <groups>
        <exclude>
            <group>slow</group>
            <group>external-api</group>
        </exclude>
    </groups>
</phpunit>
```

With this configuration, running `./vendor/bin/phpunit` skips all `slow` and `external-api` tests unless you explicitly pass `--group slow`.

## Common Pitfalls
1. **Over-grouping** — Creating dozens of hyper-specific groups makes the system hard to remember. Stick to a small set of well-known groups like `database`, `slow`, `external-api`, and `smoke`.
2. **Using `--filter` for permanent CI configuration** — Filters are fragile because they depend on class and method names. Use groups and test suites for CI; reserve `--filter` for ad-hoc local runs.

## Best Practices
1. **Document your groups** — List the group names and their meaning in your project's contributing guide. New team members should know what `--group smoke` runs.
2. **Combine suites and groups** — Suites separate by test type (Unit/Integration/Feature), groups separate by concern (database, slow, external). Together they give you precise control.

## Summary
- Use `#[Group('name')]` to tag tests by concern at the class or method level.
- Run subsets with `--group` and exclude others with `--exclude-group`.
- Use `--filter` for ad-hoc local filtering by class or method name.
- Configure default exclusions in `phpunit.xml` to keep everyday runs fast.

## Code Examples

**Class-level and method-level #[Group] attributes — the second test belongs to both 'external-api' and 'slow' groups**

```php
<?php

declare(strict_types=1);

namespace Tests\Integration;

use PHPUnit\Framework\Attributes\Group;
use PHPUnit\Framework\Attributes\Test;
use PHPUnit\Framework\TestCase;
use App\Service\EmailNotifier;

#[Group('external-api')]
class EmailNotifierTest extends TestCase
{
    #[Test]
    public function sendsWelcomeEmail(): void
    {
        $notifier = new EmailNotifier();

        $result = $notifier->sendWelcome('user@example.com');

        $this->assertTrue($result->wasSent);
    }

    #[Test]
    #[Group('slow')]
    public function retriesOnTemporaryFailure(): void
    {
        $notifier = new EmailNotifier(maxRetries: 3);

        $result = $notifier->sendWelcome('flaky@example.com');

        $this->assertTrue($result->wasSent);
        $this->assertSame(2, $result->attempts);
    }
}
```


## Resources

- [PHPUnit 12 – The Command-Line Test Runner](https://docs.phpunit.de/en/12.0/textui.html) — Official PHPUnit documentation on command-line options including --group and --filter

---

> 📘 *This lesson is part of the [PHP Testing & Quality Assurance](https://stanza.dev/courses/php-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*