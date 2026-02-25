---
source_course: "php-testing"
source_lesson: "php-testing-mocks"
---

# Mock Objects and Expectations

## Introduction
While stubs control what your dependencies return, mocks go further by verifying that your code interacts with those dependencies correctly. Mocks let you assert that a method was called, how many times it was called, and with what arguments — making them essential for testing side effects like sending emails, writing to databases, or dispatching events.

## Key Concepts
- **createMock()**: Creates a mock object that supports both stubbing and expectation verification.
- **expects()**: Sets a cardinality constraint on how many times a method should be called (e.g., `once()`, `exactly(2)`, `never()`).
- **method()**: Specifies which method the expectation applies to.
- **with()**: Constrains the arguments the method must be called with.
- **willReturn()**: Configures the mock's return value (same as stubs, but on a mock object).

## Real World Context
When your `UserService` calls `$emailSender->sendWelcomeEmail($user)`, you need to verify that the email is actually sent with the correct user data. A stub can't tell you whether `sendWelcomeEmail` was called; a mock can. This is critical for any operation where the return value doesn't matter but the side effect does.

## Deep Dive
Let's start with the most common mock pattern: verifying a method is called exactly once.

The following test ensures that registering a user triggers exactly one welcome email:

```php
<?php
declare(strict_types=1);

use PHPUnit\Framework\TestCase;

class UserServiceTest extends TestCase
{
    public function testRegisterSendsWelcomeEmail(): void
    {
        $emailSender = $this->createMock(EmailSenderInterface::class);

        // Expect sendWelcomeEmail to be called exactly once
        $emailSender->expects($this->once())
            ->method('sendWelcomeEmail')
            ->with($this->isInstanceOf(User::class));

        $repository = $this->createStub(UserRepositoryInterface::class);
        $repository->method('save')->willReturn(true);

        $service = new UserService($repository, $emailSender);
        $service->register('alice@example.com', 'Alice');

        // PHPUnit automatically verifies the expectation after the test
    }
}
```

PHPUnit verifies expectations at the end of the test automatically. If `sendWelcomeEmail` is never called, or called more than once, the test fails.

You can use different cardinality matchers depending on the scenario:

```php
<?php
// Called exactly once
$mock->expects($this->once())->method('save');

// Called exactly N times
$mock->expects($this->exactly(3))->method('log');

// Never called (verify absence of interaction)
$mock->expects($this->never())->method('delete');

// Called at least once
$mock->expects($this->atLeastOnce())->method('notify');

// Called any number of times (no verification, like a stub)
$mock->expects($this->any())->method('getName')->willReturn('Alice');
```

The snippet above shows the full range of cardinality options. Use `once()` for most cases, `never()` to guard against unwanted calls, and `exactly(N)` when the count matters.

To verify arguments, use `with()` and PHPUnit's constraint matchers:

```php
<?php
public function testOrderNotificationIncludesOrderId(): void
{
    $notifier = $this->createMock(NotificationService::class);

    $notifier->expects($this->once())
        ->method('send')
        ->with(
            $this->equalTo('order.placed'),
            $this->callback(function (array $payload): bool {
                return isset($payload['orderId'])
                    && is_int($payload['orderId'])
                    && $payload['orderId'] > 0;
            })
        );

    $orderService = new OrderService($notifier);
    $orderService->placeOrder(['item' => 'Widget', 'quantity' => 2]);
}
```

The `callback()` constraint gives you full control over argument validation when simple equality isn't enough.

When you need a mock to behave differently on consecutive calls, use `willReturnCallback()` with a counter:

```php
<?php
public function testRetriesOnFailure(): void
{
    $httpClient = $this->createMock(HttpClientInterface::class);

    // First call throws, second call returns a response
    $callCount = 0;
    $httpClient->expects($this->exactly(2))
        ->method('get')
        ->willReturnCallback(function () use (&$callCount) {
            if (++$callCount === 1) {
                throw new \RuntimeException('Timeout');
            }
            return new Response(200, '{"status": "ok"}');
        });

    $service = new ResilientApiClient($httpClient, maxRetries: 1);
    $result = $service->fetch('/api/health');

    $this->assertSame(200, $result->statusCode);
}
```

This approach uses a callback to throw on the first call and return a value on the second. It replaces the old `withConsecutive()` method that was removed in PHPUnit 10. For more complex scenarios where you need to verify different arguments on each call, combine `exactly(N)` with `willReturnCallback()`:

```php
<?php
public function testLogsEachStepWithCorrectLevel(): void
{
    $logger = $this->createMock(LoggerInterface::class);

    $callIndex = 0;
    $expectedCalls = [
        ['info', 'Processing started'],
        ['debug', 'Step 1 complete'],
        ['info', 'Processing finished'],
    ];

    $logger->expects($this->exactly(3))
        ->method('log')
        ->willReturnCallback(
            function (string $level, string $message) use (&$callIndex, $expectedCalls): void {
                $this->assertSame($expectedCalls[$callIndex][0], $level);
                $this->assertSame($expectedCalls[$callIndex][1], $message);
                $callIndex++;
            }
        );

    $processor = new DataProcessor($logger);
    $processor->run();
}
```

This approach tracks the call index manually and asserts arguments within the callback, giving you the same verification power as the removed `withConsecutive()` without relying on deprecated APIs.

PHP 8.5 introduces the `#[\NoDiscard]` attribute, which marks return values that should not be ignored. When testing code that uses `#[\NoDiscard]`, your mocks naturally validate that return values are being consumed because the mock's `willReturn()` value flows through the same code path as production.

## Common Pitfalls
1. **Verifying too many interactions** — Mocking every method call makes tests fragile. If you refactor internal method call order, all your tests break even though the behavior is correct. Only mock methods that represent meaningful side effects.
2. **Forgetting that mock expectations are auto-verified** — You don't need to call a separate verify method. PHPUnit checks expectations in its `tearDown`. If your test ends early (e.g., due to an unexpected exception), the expectation failure might be masked by the exception message.

## Best Practices
1. **One mock expectation per test** — Each test should verify one interaction. If you need to verify that `save()` AND `notify()` are called, consider splitting into two tests. This makes failures easier to diagnose.
2. **Combine stubs and mocks in the same test** — Use `createStub()` for dependencies that only provide data and `createMock()` for the one dependency whose interaction you are verifying. This communicates intent clearly.

## Summary
- `createMock()` creates a double that supports expectations via `expects()`.
- Use `once()`, `exactly(N)`, `never()`, and `atLeastOnce()` for cardinality constraints.
- `with()` validates the arguments a method receives.
- `willReturnCallback()` with a counter replaces the removed `withConsecutive()` for sequential return values.
- For argument verification on consecutive calls, use `willReturnCallback()` with a call index counter.
- PHPUnit auto-verifies mock expectations at the end of each test.

## Code Examples

**Verifying consecutive method calls with different arguments using willReturnCallback() — the modern replacement for the removed withConsecutive()**

```php
<?php
declare(strict_types=1);

use PHPUnit\Framework\TestCase;

interface EventDispatcherInterface
{
    public function dispatch(string $event, array $payload): void;
}

class CheckoutServiceTest extends TestCase
{
    public function testCheckoutDispatchesCorrectEvents(): void
    {
        $dispatcher = $this->createMock(EventDispatcherInterface::class);

        $callIndex = 0;
        $expectedEvents = [
            ['order.created', ['orderId']],
            ['payment.processed', ['orderId', 'amount']],
            ['email.queued', ['to', 'template']],
        ];

        $dispatcher->expects($this->exactly(3))
            ->method('dispatch')
            ->willReturnCallback(
                function (string $event, array $payload) use (&$callIndex, $expectedEvents): void {
                    [$expectedEvent, $expectedKeys] = $expectedEvents[$callIndex];
                    $this->assertSame($expectedEvent, $event);
                    foreach ($expectedKeys as $key) {
                        $this->assertArrayHasKey($key, $payload);
                    }
                    $callIndex++;
                }
            );

        $checkout = new CheckoutService($dispatcher);
        $checkout->process(orderId: 42, amount: 99.99, email: 'buyer@example.com');
    }
}
?>
```


## Resources

- [PHPUnit Mock Objects Documentation](https://docs.phpunit.de/en/12.0/test-doubles.html#mock-objects) — Official PHPUnit 12 documentation on mock objects and expectations
- [PHP 8.5 NoDiscard Attribute](https://www.php.net/releases/8.5/en.php) — PHP 8.5 release page covering the #[\NoDiscard] attribute and other new features

---

> 📘 *This lesson is part of the [PHP Testing & Quality Assurance](https://stanza.dev/courses/php-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*