---
source_course: "laravel-testing"
source_lesson: "laravel-testing-testing-value-objects"
---

# Testing Value Objects and DTOs

Value Objects and Data Transfer Objects (DTOs) are perfect candidates for unit testing because they contain pure business logic without dependencies.

## What are Value Objects?

Value Objects are immutable objects that represent a value and are compared by their values, not their identity.

```php
// app/ValueObjects/Money.php
final class Money
{
    public function __construct(
        private readonly int $cents,
        private readonly string $currency = 'USD'
    ) {
        if ($cents < 0) {
            throw new \InvalidArgumentException('Amount cannot be negative');
        }
    }

    public static function fromDollars(float $dollars, string $currency = 'USD'): self
    {
        return new self((int) round($dollars * 100), $currency);
    }

    public function cents(): int
    {
        return $this->cents;
    }

    public function dollars(): float
    {
        return $this->cents / 100;
    }

    public function currency(): string
    {
        return $this->currency;
    }

    public function add(Money $other): self
    {
        $this->ensureSameCurrency($other);
        return new self($this->cents + $other->cents, $this->currency);
    }

    public function subtract(Money $other): self
    {
        $this->ensureSameCurrency($other);
        return new self($this->cents - $other->cents, $this->currency);
    }

    public function multiply(float $multiplier): self
    {
        return new self((int) round($this->cents * $multiplier), $this->currency);
    }

    public function equals(Money $other): bool
    {
        return $this->cents === $other->cents
            && $this->currency === $other->currency;
    }

    public function format(): string
    {
        $symbols = ['USD' => '$', 'EUR' => '€', 'GBP' => '£'];
        $symbol = $symbols[$this->currency] ?? $this->currency . ' ';
        return $symbol . number_format($this->dollars(), 2);
    }

    private function ensureSameCurrency(Money $other): void
    {
        if ($this->currency !== $other->currency) {
            throw new \InvalidArgumentException(
                "Cannot operate on different currencies: {$this->currency} and {$other->currency}"
            );
        }
    }
}
```

Comprehensive tests:

```php
<?php

namespace Tests\Unit\ValueObjects;

use App\ValueObjects\Money;
use PHPUnit\Framework\TestCase;

class MoneyTest extends TestCase
{
    // Creation Tests
    public function test_creates_from_cents(): void
    {
        $money = new Money(1000);

        $this->assertEquals(1000, $money->cents());
        $this->assertEquals(10.00, $money->dollars());
    }

    public function test_creates_from_dollars(): void
    {
        $money = Money::fromDollars(10.99);

        $this->assertEquals(1099, $money->cents());
    }

    public function test_handles_rounding_from_dollars(): void
    {
        $money = Money::fromDollars(10.999);

        $this->assertEquals(1100, $money->cents()); // Rounds up
    }

    public function test_throws_for_negative_amount(): void
    {
        $this->expectException(\InvalidArgumentException::class);
        $this->expectExceptionMessage('Amount cannot be negative');

        new Money(-100);
    }

    // Arithmetic Tests
    public function test_adds_money(): void
    {
        $a = Money::fromDollars(10.00);
        $b = Money::fromDollars(5.50);

        $result = $a->add($b);

        $this->assertEquals(1550, $result->cents());
    }

    public function test_subtracts_money(): void
    {
        $a = Money::fromDollars(10.00);
        $b = Money::fromDollars(3.50);

        $result = $a->subtract($b);

        $this->assertEquals(650, $result->cents());
    }

    public function test_multiplies_money(): void
    {
        $money = Money::fromDollars(10.00);

        $result = $money->multiply(1.5);

        $this->assertEquals(1500, $result->cents());
    }

    public function test_arithmetic_is_immutable(): void
    {
        $original = Money::fromDollars(10.00);
        $result = $original->add(Money::fromDollars(5.00));

        $this->assertEquals(1000, $original->cents()); // Unchanged
        $this->assertEquals(1500, $result->cents());
    }

    // Currency Tests
    public function test_defaults_to_usd(): void
    {
        $money = new Money(1000);

        $this->assertEquals('USD', $money->currency());
    }

    public function test_accepts_different_currencies(): void
    {
        $eur = new Money(1000, 'EUR');

        $this->assertEquals('EUR', $eur->currency());
    }

    public function test_throws_when_adding_different_currencies(): void
    {
        $this->expectException(\InvalidArgumentException::class);
        $this->expectExceptionMessage('Cannot operate on different currencies');

        $usd = Money::fromDollars(10.00, 'USD');
        $eur = Money::fromDollars(10.00, 'EUR');

        $usd->add($eur);
    }

    // Equality Tests
    public function test_equals_same_amount_and_currency(): void
    {
        $a = Money::fromDollars(10.00);
        $b = Money::fromDollars(10.00);

        $this->assertTrue($a->equals($b));
    }

    public function test_not_equals_different_amount(): void
    {
        $a = Money::fromDollars(10.00);
        $b = Money::fromDollars(20.00);

        $this->assertFalse($a->equals($b));
    }

    public function test_not_equals_different_currency(): void
    {
        $a = Money::fromDollars(10.00, 'USD');
        $b = Money::fromDollars(10.00, 'EUR');

        $this->assertFalse($a->equals($b));
    }

    // Formatting Tests
    public function test_formats_usd(): void
    {
        $money = Money::fromDollars(1234.56);

        $this->assertEquals('$1,234.56', $money->format());
    }

    public function test_formats_eur(): void
    {
        $money = Money::fromDollars(1234.56, 'EUR');

        $this->assertEquals('€1,234.56', $money->format());
    }

    public function test_formats_unknown_currency(): void
    {
        $money = Money::fromDollars(100.00, 'JPY');

        $this->assertEquals('JPY 100.00', $money->format());
    }
}
```

## Testing DTOs

DTOs transfer data between layers:

```php
// app/DTOs/CreateUserData.php
final class CreateUserData
{
    public function __construct(
        public readonly string $name,
        public readonly string $email,
        public readonly string $password,
        public readonly ?string $phone = null,
        public readonly array $roles = ['user']
    ) {}

    public static function fromRequest(array $data): self
    {
        return new self(
            name: $data['name'],
            email: strtolower(trim($data['email'])),
            password: $data['password'],
            phone: $data['phone'] ?? null,
            roles: $data['roles'] ?? ['user']
        );
    }

    public function toArray(): array
    {
        return [
            'name' => $this->name,
            'email' => $this->email,
            'password' => $this->password,
            'phone' => $this->phone,
            'roles' => $this->roles,
        ];
    }

    public function hasRole(string $role): bool
    {
        return in_array($role, $this->roles, true);
    }
}
```

```php
<?php

namespace Tests\Unit\DTOs;

use App\DTOs\CreateUserData;
use PHPUnit\Framework\TestCase;

class CreateUserDataTest extends TestCase
{
    public function test_creates_from_request_data(): void
    {
        $data = [
            'name' => 'John Doe',
            'email' => '  JOHN@EXAMPLE.COM  ',
            'password' => 'secret123',
        ];

        $dto = CreateUserData::fromRequest($data);

        $this->assertEquals('John Doe', $dto->name);
        $this->assertEquals('john@example.com', $dto->email); // Normalized
        $this->assertEquals('secret123', $dto->password);
        $this->assertNull($dto->phone);
        $this->assertEquals(['user'], $dto->roles);
    }

    public function test_accepts_optional_fields(): void
    {
        $data = [
            'name' => 'John',
            'email' => 'john@example.com',
            'password' => 'secret',
            'phone' => '+1234567890',
            'roles' => ['admin', 'editor'],
        ];

        $dto = CreateUserData::fromRequest($data);

        $this->assertEquals('+1234567890', $dto->phone);
        $this->assertEquals(['admin', 'editor'], $dto->roles);
    }

    public function test_converts_to_array(): void
    {
        $dto = new CreateUserData(
            name: 'John',
            email: 'john@example.com',
            password: 'secret',
        );

        $array = $dto->toArray();

        $this->assertEquals('John', $array['name']);
        $this->assertEquals('john@example.com', $array['email']);
        $this->assertArrayHasKey('phone', $array);
    }

    public function test_checks_role(): void
    {
        $dto = new CreateUserData(
            name: 'Admin',
            email: 'admin@example.com',
            password: 'secret',
            roles: ['admin', 'editor']
        );

        $this->assertTrue($dto->hasRole('admin'));
        $this->assertTrue($dto->hasRole('editor'));
        $this->assertFalse($dto->hasRole('user'));
    }
}
```

## Resources

- [Value Objects in PHP](https://martinfowler.com/bliki/ValueObject.html) — Martin Fowler's explanation of Value Objects

---

> 📘 *This lesson is part of the [Laravel Testing Mastery](https://stanza.dev/courses/laravel-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*