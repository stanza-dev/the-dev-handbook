---
source_course: "php-testing"
source_lesson: "php-testing-dependency-injection"
---

# Dependency Injection for Testability

## Introduction
Test doubles only work when you can substitute them for real dependencies. Dependency injection (DI) is the design pattern that makes this substitution possible. By passing dependencies into a class through its constructor instead of creating them internally, you gain the ability to swap real implementations for stubs and mocks during testing.

## Key Concepts
- **Dependency Injection**: A technique where an object receives its dependencies from the outside rather than creating them itself.
- **Constructor Injection**: Passing dependencies through the constructor, the most common and recommended form of DI.
- **Interface Segregation**: Depending on interfaces rather than concrete classes, which allows any implementation (real or test double) to be injected.
- **Inversion of Control**: The principle that a class should not control the creation of its dependencies — that responsibility belongs to the caller.

## Real World Context
In legacy codebases, you often find classes that instantiate their own dependencies with `new`. These classes are nearly impossible to test in isolation because you cannot replace the internal dependency with a test double. Refactoring to use constructor injection is typically the first step in making untestable code testable.

## Deep Dive
Consider a class that creates its own dependency internally:

```php
<?php
// Hard to test: the dependency is created inside the class
class ReportGenerator
{
    public function generate(int $month): string
    {
        // Cannot replace this with a stub in tests!
        $database = new MySQLDatabase('localhost', 'reports_db');
        $data = $database->query("SELECT * FROM sales WHERE month = $month");

        return $this->format($data);
    }
}
```

This class is tightly coupled to `MySQLDatabase`. Testing `generate()` requires a real MySQL connection, making the test slow, fragile, and dependent on external state.

The fix is straightforward — inject the dependency through the constructor:

```php
<?php
// Easy to test: dependency is injected
interface DatabaseInterface
{
    public function query(string $sql): array;
}

class ReportGenerator
{
    public function __construct(
        private readonly DatabaseInterface $database
    ) {}

    public function generate(int $month): string
    {
        $data = $this->database->query(
            "SELECT * FROM sales WHERE month = :month",
        );

        return $this->format($data);
    }

    private function format(array $data): string
    {
        // ... formatting logic
        return implode(', ', array_column($data, 'product'));
    }
}
```

Now you can inject a stub in your test:

```php
<?php
class ReportGeneratorTest extends TestCase
{
    public function testGenerateFormatsDataCorrectly(): void
    {
        $database = $this->createStub(DatabaseInterface::class);
        $database->method('query')
            ->willReturn([
                ['product' => 'Widget', 'total' => 150],
                ['product' => 'Gadget', 'total' => 300],
            ]);

        $generator = new ReportGenerator($database);
        $report = $generator->generate(month: 6);

        $this->assertStringContainsString('Widget', $report);
        $this->assertStringContainsString('Gadget', $report);
    }
}
```

The test is fast, deterministic, and completely isolated from the database.

PHP's constructor promotion syntax (introduced in PHP 8.0) makes DI concise. Combined with `readonly` properties (PHP 8.1+), your constructors become clean and immutable:

```php
<?php
class OrderService
{
    public function __construct(
        private readonly OrderRepositoryInterface $orderRepository,
        private readonly PaymentGatewayInterface $paymentGateway,
        private readonly EventDispatcherInterface $eventDispatcher,
        private readonly LoggerInterface $logger,
    ) {}
}
```

Each dependency is typed as an interface, making it trivially easy to stub or mock any of them in tests.

PHP 8.5 introduces an enhancement to readonly classes: the `clone()` function with property overrides. This is particularly useful for creating test fixtures from immutable value objects:

```php
<?php
// PHP 8.5: clone() with property overrides for readonly classes
readonly class OrderItem
{
    public function __construct(
        public string $productId,
        public int $quantity,
        public float $unitPrice,
    ) {}
}

class OrderCalculatorTest extends TestCase
{
    private OrderItem $baseItem;

    protected function setUp(): void
    {
        // Create a base fixture once
        $this->baseItem = new OrderItem(
            productId: 'PROD-001',
            quantity: 1,
            unitPrice: 10.00,
        );
    }

    public function testBulkDiscount(): void
    {
        // Clone with modified quantity for this test
        $bulkItem = clone($this->baseItem, [
            'quantity' => 100,
        ]);

        $calculator = new OrderCalculator();
        $total = $calculator->calculateTotal($bulkItem);

        // Bulk discount: 100 * 10.00 * 0.9 = 900.00
        $this->assertSame(900.00, $total);
    }

    public function testPremiumPricing(): void
    {
        // Clone with modified price for this test
        $premiumItem = clone($this->baseItem, [
            'unitPrice' => 99.99,
        ]);

        $calculator = new OrderCalculator();
        $total = $calculator->calculateTotal($premiumItem);

        $this->assertSame(99.99, $total);
    }
}
```

The `clone()` with property overrides lets you derive variations from a base fixture without manually reconstructing the entire object, keeping tests concise and expressive.

## Common Pitfalls
1. **Injecting too many dependencies** — If your constructor has more than four or five parameters, the class is likely doing too much. This is a design smell that suggests the class should be split into smaller, focused classes. DI makes this smell visible, which is a feature, not a bug.
2. **Using a service locator instead of DI** — Passing a container or service locator into a class hides its real dependencies. Tests become harder to write because you need to configure the entire container. Always prefer explicit constructor parameters.

## Best Practices
1. **Type dependencies as interfaces** — This is the single most impactful practice for testability. An interface-typed parameter accepts any implementation, including stubs and mocks, without any special setup.
2. **Make dependencies readonly** — Use `private readonly` in constructor promotion. This prevents accidental reassignment and communicates that dependencies are set once at construction time and never change.

## Summary
- Constructor injection is the foundation of testable code — it lets you swap real dependencies for test doubles.
- Type dependencies as interfaces so stubs and mocks can be injected seamlessly.
- PHP 8.0+ constructor promotion and `readonly` properties make DI syntax clean and concise.
- PHP 8.5's `clone()` with property overrides simplifies creating variant test fixtures from immutable value objects.
- If a class has too many injected dependencies, it's a signal to refactor into smaller classes.

## Code Examples

**Refactoring from hard-coded dependency to constructor injection, then testing with a stub that simulates partial delivery failure**

```php
<?php
declare(strict_types=1);

// Before: untestable — creates its own dependency
class NewsletterSender
{
    public function send(array $subscribers): int
    {
        $mailer = new SmtpMailer('smtp.example.com', 587);
        $sent = 0;
        foreach ($subscribers as $email) {
            if ($mailer->send($email, 'Newsletter', '...')) {
                $sent++;
            }
        }
        return $sent;
    }
}

// After: testable — dependency injected through constructor
interface MailerInterface
{
    public function send(string $to, string $subject, string $body): bool;
}

class NewsletterSender
{
    public function __construct(
        private readonly MailerInterface $mailer
    ) {}

    public function send(array $subscribers, string $subject, string $body): int
    {
        $sent = 0;
        foreach ($subscribers as $email) {
            if ($this->mailer->send($email, $subject, $body)) {
                $sent++;
            }
        }
        return $sent;
    }
}

// Test: inject a stub to control mailer behavior
class NewsletterSenderTest extends TestCase
{
    public function testCountsSuccessfulSends(): void
    {
        $mailer = $this->createStub(MailerInterface::class);
        // First two succeed, third fails
        $callCount = 0;
        $mailer->method('send')
            ->willReturnCallback(function () use (&$callCount): bool {
                return ++$callCount <= 2;
            });

        $sender = new NewsletterSender($mailer);
        $count = $sender->send(
            ['a@test.com', 'b@test.com', 'c@test.com'],
            'News',
            'Hello!'
        );

        $this->assertSame(2, $count);
    }
}
?>
```


## Resources

- [PHP Constructor Promotion](https://www.php.net/manual/en/language.oop5.decon.php#language.oop5.decon.constructor.promotion) — Official PHP documentation on constructor property promotion
- [PHPUnit Test Doubles Documentation](https://docs.phpunit.de/en/12.0/test-doubles.html) — Official PHPUnit 12 guide to creating stubs and mocks for injected dependencies

---

> 📘 *This lesson is part of the [PHP Testing & Quality Assurance](https://stanza.dev/courses/php-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*