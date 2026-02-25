---
source_course: "php-testing"
source_lesson: "php-testing-exceptions"
---

# Testing Exceptions and Errors

## Introduction
Good code throws exceptions when something goes wrong. Good tests verify that those exceptions are thrown with the correct type, message, and code. PHPUnit provides dedicated methods — `expectException()`, `expectExceptionMessage()`, and `expectExceptionCode()` — to assert that a block of code throws exactly the exception you expect.

## Key Concepts
- **expectException()**: Tells PHPUnit that the test must throw an exception of the given class, or the test fails.
- **expectExceptionMessage()**: Asserts the exception message contains (or exactly matches) a given string.
- **expectExceptionCode()**: Asserts the exception's numeric or string error code.

## Real World Context
An API endpoint that accepts a user ID should throw a `NotFoundException` when the ID does not exist. If your test does not verify this, a refactor could accidentally swallow the exception or throw a generic `\Exception`, and your test suite would stay green while your API returns 500 errors instead of 404s.

## Deep Dive

### Basic Exception Testing

Call the `expect*` methods *before* the code that throws. PHPUnit sets up an internal expectation and catches the exception automatically:

```php
<?php

declare(strict_types=1);

namespace Tests\Unit;

use PHPUnit\Framework\Attributes\Test;
use PHPUnit\Framework\TestCase;
use App\UserService;
use App\Exception\UserNotFoundException;

class UserServiceTest extends TestCase
{
    #[Test]
    public function throwsWhenUserNotFound(): void
    {
        $userService = new UserService();

        $this->expectException(UserNotFoundException::class);
        $this->expectExceptionMessage('User with ID 999 not found');
        $this->expectExceptionCode(404);

        // This line must throw — if it does not, the test FAILS
        $userService->findOrFail(999);
    }
}
```

The three `expect*` calls must come *before* the throwing code. If you place them after, they are never reached.

### Testing Exception Type Only

When you only care about the exception class and not its message or code, a single `expectException()` is enough:

```php
<?php

#[Test]
public function rejectsDivisionByZero(): void
{
    $calculator = new Calculator();

    $this->expectException(\DivisionByZeroError::class);

    $calculator->divide(10, 0);
}
```

This is the simplest form of exception testing and covers the majority of cases.

### Testing Exception Message with Substring

`expectExceptionMessage()` checks that the exception message *contains* the given string (it does not have to be an exact match):

```php
<?php

#[Test]
public function rejectsNegativeQuantity(): void
{
    $orderService = new OrderService();

    $this->expectException(\InvalidArgumentException::class);
    $this->expectExceptionMessage('Quantity must be positive');

    // Throws: "Quantity must be positive, got -3"
    $orderService->addItem($productId, quantity: -3);
}
```

The test passes because the actual message contains the expected substring.

### Multiple Exception Tests in One Class

Each test method can only expect one exception. If you need to test multiple exception scenarios, use separate test methods or a data provider:

```php
<?php

#[Test]
#[DataProvider('invalidInputs')]
public function rejectsInvalidInput(mixed $input, string $expectedMessage): void
{
    $validator = new InputValidator();

    $this->expectException(\InvalidArgumentException::class);
    $this->expectExceptionMessage($expectedMessage);

    $validator->validate($input);
}

public static function invalidInputs(): array
{
    return [
        'empty string' => ['', 'Input cannot be empty'],
        'too long'     => [str_repeat('a', 256), 'Input exceeds 255 characters'],
        'null value'   => [null, 'Input cannot be null'],
    ];
}
```

This pattern keeps exception tests concise while covering every edge case.

## Common Pitfalls
1. **Placing `expect*` calls after the throwing code** — The exception is thrown before PHPUnit registers the expectation, so the test fails with an unexpected exception instead of passing. Always set expectations first.
2. **Testing for the base `\Exception` class** — This is too broad. Your code might throw a different exception type by accident and the test would still pass. Always test for the specific exception class.

## Best Practices
1. **Use custom exception classes** — Create domain-specific exceptions like `InsufficientFundsException` instead of throwing generic `\RuntimeException`. This makes exception tests precise and your codebase more expressive.
2. **Combine with data providers for validation** — When a method has multiple rejection conditions, use a data provider to test each one. This avoids an explosion of nearly identical test methods.

## Summary
- Call `expectException()`, `expectExceptionMessage()`, and `expectExceptionCode()` *before* the code that throws.
- Each test method can only test one exception scenario.
- Use specific exception classes, not generic `\Exception`.
- Combine exception testing with data providers for thorough validation coverage.

## Code Examples

**Testing that a bank account throws InsufficientFundsException on overdraft, using all three expect methods**

```php
<?php

declare(strict_types=1);

namespace Tests\Unit;

use PHPUnit\Framework\Attributes\DataProvider;
use PHPUnit\Framework\Attributes\Test;
use PHPUnit\Framework\TestCase;
use App\BankAccount;
use App\Exception\InsufficientFundsException;

class BankAccountTest extends TestCase
{
    #[Test]
    public function throwsOnOverdraft(): void
    {
        $account = new BankAccount(balance: 100.00);

        $this->expectException(InsufficientFundsException::class);
        $this->expectExceptionMessage('Cannot withdraw 150.00');
        $this->expectExceptionCode(422);

        $account->withdraw(150.00);
    }

    #[Test]
    public function allowsWithdrawalWithinBalance(): void
    {
        $account = new BankAccount(balance: 100.00);

        $account->withdraw(60.00);

        $this->assertSame(40.00, $account->balance());
    }
}
```


## Resources

- [PHPUnit 12 – Testing Exceptions](https://docs.phpunit.de/en/12.0/writing-tests-for-phpunit.html#testing-exceptions) — Official PHPUnit documentation on expectException and related methods

---

> 📘 *This lesson is part of the [PHP Testing & Quality Assurance](https://stanza.dev/courses/php-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*