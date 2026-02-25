---
source_course: "php-testing"
source_lesson: "php-testing-setup-teardown"
---

# Test Lifecycle: Setup and Teardown

## Introduction
PHPUnit provides lifecycle hooks that run at specific points before and after your tests. These hooks let you create shared fixtures, open database connections, or reset global state without duplicating code in every test method. Understanding the lifecycle is essential for writing clean, maintainable test suites.

## Key Concepts
- **setUp()**: An instance method called before *each* test method in the class. Use it to create fresh objects your tests need.
- **tearDown()**: An instance method called after *each* test method. Use it to clean up resources like file handles or database transactions.
- **setUpBeforeClass()**: A *static* method called once before the first test in the class. Use it for expensive one-time setup like loading a large fixture file.
- **tearDownAfterClass()**: A *static* method called once after the last test in the class. Use it to release shared resources.

## Real World Context
Imagine a test class with 20 test methods that all need a `UserRepository` instance. Without `setUp()`, you would construct the repository 20 times in 20 methods. With `setUp()`, you write the construction once and every test receives a fresh instance automatically, guaranteeing test isolation.

## Deep Dive

### The Full Lifecycle Order

When PHPUnit runs a test class with two test methods, the execution order is:

```
setUpBeforeClass()        ← once, before any test
  setUp()                 ← before first test
    testMethodOne()
  tearDown()              ← after first test
  setUp()                 ← before second test
    testMethodTwo()
  tearDown()              ← after second test
tearDownAfterClass()      ← once, after all tests
```

This order guarantees that each test starts with a clean slate via `setUp()` while allowing one-time initialization in `setUpBeforeClass()`.

### Using setUp() for Fresh Fixtures

The most common lifecycle hook is `setUp()`. Store shared objects as class properties:

```php
<?php

declare(strict_types=1);

namespace Tests\Unit;

use PHPUnit\Framework\Attributes\Test;
use PHPUnit\Framework\TestCase;
use App\ShoppingCart;
use App\Product;

class ShoppingCartTest extends TestCase
{
    private ShoppingCart $cart;
    private Product $defaultProduct;

    protected function setUp(): void
    {
        $this->cart = new ShoppingCart();
        $this->defaultProduct = new Product(
            name: 'Wireless Mouse',
            price: 29.99
        );
    }

    #[Test]
    public function startsEmpty(): void
    {
        $this->assertCount(0, $this->cart->items());
        $this->assertSame(0.0, $this->cart->total());
    }

    #[Test]
    public function addsProductToCart(): void
    {
        $this->cart->add($this->defaultProduct, quantity: 2);

        $this->assertCount(1, $this->cart->items());
        $this->assertSame(59.98, $this->cart->total());
    }
}
```

Each test gets its own `ShoppingCart` instance because `setUp()` runs before every test method. The second test cannot be affected by the first.

### Using tearDown() for Cleanup

`tearDown()` runs after each test method, even if the test fails. Use it to release resources:

```php
<?php

protected function tearDown(): void
{
    // Close file handles, rollback transactions, etc.
    if (isset($this->tempFile) && file_exists($this->tempFile)) {
        unlink($this->tempFile);
    }
}
```

This ensures temporary files do not accumulate across test runs.

### Static Hooks for Expensive Setup

`setUpBeforeClass()` and `tearDownAfterClass()` are static and run only once per class. They are ideal for expensive operations like loading a fixture file:

```php
<?php

private static array $fixtureData;

public static function setUpBeforeClass(): void
{
    // Load a large CSV fixture once instead of per-test
    self::$fixtureData = CsvParser::parse(__DIR__ . '/fixtures/products.csv');
}

public static function tearDownAfterClass(): void
{
    self::$fixtureData = [];
}
```

Because these methods are static, they cannot access `$this`. Store shared data in static properties instead.

## Common Pitfalls
1. **Sharing mutable state via `setUpBeforeClass()`** — If one test modifies a static fixture array, subsequent tests see the mutation. Only use static hooks for read-only data or reset the state in `setUp()`.
2. **Forgetting that `setUp()` must call `parent::setUp()`** — If you extend a custom base test class that has its own `setUp()`, you must call `parent::setUp()` to avoid breaking the parent's initialization.

## Best Practices
1. **Prefer `setUp()` over `setUpBeforeClass()`** — Per-test setup guarantees isolation. Only use the class-level hook when the setup cost is genuinely prohibitive (e.g., spinning up a test container).
2. **Keep `tearDown()` idempotent** — It should be safe to run even if `setUp()` partially failed. Always check that a resource exists before trying to release it.

## Summary
- `setUp()` runs before each test; `tearDown()` runs after each test.
- `setUpBeforeClass()` runs once before the first test; `tearDownAfterClass()` runs once after the last.
- Use `setUp()` for fresh per-test fixtures and `tearDown()` for cleanup.
- Static hooks are for expensive, read-only shared resources.

## Code Examples

**Shows setUp() creating a temp directory before each test and tearDown() cleaning it up after, ensuring full test isolation**

```php
<?php

declare(strict_types=1);

namespace Tests\Unit;

use PHPUnit\Framework\Attributes\Test;
use PHPUnit\Framework\TestCase;
use App\TemporaryFileManager;

class TemporaryFileManagerTest extends TestCase
{
    private TemporaryFileManager $manager;
    private string $tempDir;

    protected function setUp(): void
    {
        $this->tempDir = sys_get_temp_dir() . '/phpunit_test_' . uniqid();
        mkdir($this->tempDir);
        $this->manager = new TemporaryFileManager($this->tempDir);
    }

    protected function tearDown(): void
    {
        // Clean up temp directory after each test
        array_map('unlink', glob($this->tempDir . '/*'));
        if (is_dir($this->tempDir)) {
            rmdir($this->tempDir);
        }
    }

    #[Test]
    public function createsTemporaryFile(): void
    {
        $path = $this->manager->create('report.txt', 'Hello');

        $this->assertFileExists($path);
        $this->assertSame('Hello', file_get_contents($path));
    }

    #[Test]
    public function deletesTemporaryFile(): void
    {
        $path = $this->manager->create('old.txt', 'data');
        $this->manager->delete('old.txt');

        $this->assertFileDoesNotExist($path);
    }
}
```


## Resources

- [PHPUnit 12 – Fixtures](https://docs.phpunit.de/en/12.0/fixtures.html) — Official PHPUnit documentation on setUp, tearDown, and test fixtures

---

> 📘 *This lesson is part of the [PHP Testing & Quality Assurance](https://stanza.dev/courses/php-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*