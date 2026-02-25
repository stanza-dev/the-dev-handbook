---
source_course: "php-testing"
source_lesson: "php-testing-test-doubles"
---

# Understanding Test Doubles

## Introduction
When unit testing, you need to isolate the class under test from its dependencies. Test doubles are stand-in objects that replace real collaborators during testing, letting you control inputs and verify interactions without triggering side effects like database writes or API calls.

## Key Concepts
- **Test Double**: A generic term for any object that stands in for a real dependency during a test.
- **Dummy**: An object passed to satisfy a parameter requirement but never actually used. It exists only to fill a type hint.
- **Stub**: An object that returns pre-configured answers to method calls. It controls indirect inputs to the system under test.
- **Mock**: An object that records calls and verifies expectations. It checks that the system under test interacts correctly with its collaborators.
- **Spy**: A variation that records all calls for later inspection, rather than declaring expectations upfront.
- **Fake**: A working implementation with shortcuts, such as an in-memory repository instead of a database-backed one.

## Real World Context
Imagine you are testing an `OrderService` that depends on a `PaymentGateway` and a `NotificationService`. Without test doubles, every test would charge a real credit card and send a real email. Test doubles let you replace those dependencies with controlled substitutes so you can test the order logic in complete isolation.

## Deep Dive
The five types of test doubles form a spectrum from passive to active. Understanding which type to use in each situation prevents over-specification (brittle tests) and under-specification (tests that miss bugs).

Here is a summary table showing when to reach for each type of double:

```php
<?php
// Dummy: passed but never called
// Used when a constructor requires an argument you don't care about
$dummyLogger = $this->createStub(LoggerInterface::class);
$service = new ImportService($dummyLogger);

// Stub: returns canned data
$priceStub = $this->createStub(PricingService::class);
$priceStub->method('getPrice')->willReturn(19.99);

// Mock: verifies interaction
$notifierMock = $this->createMock(NotificationService::class);
$notifierMock->expects($this->once())->method('send');

// Spy: records calls for later assertions
// (PHPUnit does not have a built-in spy, but mocks can act as spies
//  by using ->expects($this->any()) and inspecting after the act phase)

// Fake: a lightweight real implementation
// e.g., InMemoryUserRepository implements UserRepositoryInterface
```

The code above shows the most common patterns in a single glance. Notice that dummies and stubs use `createStub()`, while mocks use `createMock()`. The distinction matters because stubs should never fail a test on their own — they only provide data — whereas mocks carry expectations that cause test failures if not met.

A practical guideline is the **command-query separation** principle applied to test doubles. Use stubs for queries (methods that return data) and mocks for commands (methods that cause side effects).

Here is a more complete example that puts all the pieces together:

```php
<?php
declare(strict_types=1);

use PHPUnit\Framework\TestCase;

class OrderServiceTest extends TestCase
{
    public function testPlaceOrderChargesPaymentAndNotifiesCustomer(): void
    {
        // Stub: controls the pricing lookup (query)
        $catalog = $this->createStub(ProductCatalog::class);
        $catalog->method('getPrice')
            ->willReturn(49.99);

        // Mock: verifies the payment is charged (command)
        $payment = $this->createMock(PaymentGateway::class);
        $payment->expects($this->once())
            ->method('charge')
            ->with(49.99);

        // Mock: verifies a notification is sent (command)
        $notifier = $this->createMock(NotificationService::class);
        $notifier->expects($this->once())
            ->method('sendOrderConfirmation');

        $service = new OrderService($catalog, $payment, $notifier);
        $service->placeOrder('PROD-001', quantity: 1);
    }
}
```

In this test, the `ProductCatalog` is a stub because we only need it to return a price. The `PaymentGateway` and `NotificationService` are mocks because we need to verify they were called correctly. This separation keeps the test focused and avoids unnecessary coupling.

## Common Pitfalls
1. **Mocking everything** — Over-mocking leads to tests that mirror the implementation rather than testing behavior. Only mock collaborators that cross an architectural boundary (I/O, network, file system). Value objects and simple data transformations should use real instances.
2. **Using mocks when stubs suffice** — If you only need a dependency to return data, use `createStub()`. Reaching for `createMock()` and `expects()` when you don't need interaction verification makes tests brittle and harder to read.

## Best Practices
1. **Follow command-query separation** — Stub queries, mock commands. This keeps tests focused on what matters: data correctness for queries and correct side effects for commands.
2. **Program to interfaces** — Test doubles work best when dependencies are typed as interfaces rather than concrete classes. This makes swapping real implementations for doubles natural and reinforces good design.

## Summary
- Test doubles replace real dependencies so you can test in isolation.
- The five types are dummy, stub, mock, spy, and fake — each serves a different purpose.
- Use stubs to control inputs (queries) and mocks to verify interactions (commands).
- Prefer interfaces over concrete classes so dependencies are easy to double.
- Avoid over-mocking — only mock collaborators that involve I/O or side effects.

## Code Examples

**A fake repository implementation that stores data in memory, demonstrating how fakes provide real behavior without external dependencies**

```php
<?php
declare(strict_types=1);

// Fake: a working in-memory implementation of a repository interface
interface UserRepositoryInterface
{
    public function findById(int $id): ?User;
    public function save(User $user): void;
}

class InMemoryUserRepository implements UserRepositoryInterface
{
    /** @var array<int, User> */
    private array $users = [];

    public function findById(int $id): ?User
    {
        return $this->users[$id] ?? null;
    }

    public function save(User $user): void
    {
        $this->users[$user->id] = $user;
    }
}

// Usage in a test
class UserServiceTest extends TestCase
{
    public function testRegisterUser(): void
    {
        // Use the fake instead of a database-backed repository
        $repository = new InMemoryUserRepository();
        $service = new UserService($repository);

        $user = $service->register('alice@example.com', 'Alice');

        // Assert against the fake's state
        $stored = $repository->findById($user->id);
        $this->assertNotNull($stored);
        $this->assertSame('alice@example.com', $stored->email);
    }
}
?>
```


## Resources

- [PHPUnit Test Doubles Documentation](https://docs.phpunit.de/en/12.0/test-doubles.html) — Official PHPUnit 12 documentation on stubs, mocks, and test doubles
- [Martin Fowler — Mocks Aren't Stubs](https://martinfowler.com/articles/mocksArentStubs.html) — Foundational article explaining the differences between test double types

---

> 📘 *This lesson is part of the [PHP Testing & Quality Assurance](https://stanza.dev/courses/php-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*