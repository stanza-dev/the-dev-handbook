---
source_course: "laravel-testing"
source_lesson: "laravel-testing-unit-test-fundamentals"
---

# Unit Test Fundamentals

Unit tests verify that individual pieces of code work correctly in isolation. They're the foundation of a solid test suite.

## What is a Unit Test?

A unit test:
- Tests a single "unit" (class, method, or function)
- Runs in complete isolation from other code
- Doesn't touch the database, filesystem, or external services
- Is extremely fast (milliseconds)

```
┌─────────────────────────────────────────────────────────┐
│                    Your Application                     │
├─────────────────────────────────────────────────────────┤
│                                                         │
│    ┌──────────┐   ┌──────────┐   ┌──────────┐          │
│    │ Service  │   │   Model  │   │  Helper  │          │
│    │  Class   │   │  Logic   │   │ Function │          │
│    └──────────┘   └──────────┘   └──────────┘          │
│         ▲              ▲              ▲                 │
│         │              │              │                 │
│    Unit Test 1    Unit Test 2    Unit Test 3           │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

## Creating Unit Tests

```bash
php artisan make:test UserTest --unit
```

This creates `tests/Unit/UserTest.php`:

```php
<?php

namespace Tests\Unit;

use PHPUnit\Framework\TestCase;  // Note: PHPUnit, not Laravel's TestCase

class UserTest extends TestCase
{
    // Your tests here
}
```

**Important**: Unit tests extend `PHPUnit\Framework\TestCase`, NOT `Tests\TestCase`. This means Laravel's application isn't booted, keeping tests fast and isolated.

## Testing a Model's Business Logic

Consider a `User` model with business logic:

```php
// app/Models/User.php
class User extends Model
{
    protected function fullName(): Attribute
    {
        return Attribute::make(
            get: fn () => trim("{$this->first_name} {$this->last_name}")
        );
    }

    public function isAdmin(): bool
    {
        return $this->role === 'admin';
    }

    public function age(): int
    {
        return $this->birth_date->age;
    }

    public function canAccessPremiumContent(): bool
    {
        return $this->isAdmin() || $this->subscription_tier === 'premium';
    }
}
```

Unit tests for this model:

```php
<?php

namespace Tests\Unit;

use App\Models\User;
use Carbon\Carbon;
use PHPUnit\Framework\TestCase;

class UserTest extends TestCase
{
    public function test_full_name_combines_first_and_last_name(): void
    {
        $user = new User([
            'first_name' => 'John',
            'last_name' => 'Doe',
        ]);

        $this->assertEquals('John Doe', $user->full_name);
    }

    public function test_full_name_handles_missing_last_name(): void
    {
        $user = new User(['first_name' => 'John']);

        $this->assertEquals('John', $user->full_name);
    }

    public function test_is_admin_returns_true_for_admin_role(): void
    {
        $user = new User(['role' => 'admin']);

        $this->assertTrue($user->isAdmin());
    }

    public function test_is_admin_returns_false_for_other_roles(): void
    {
        $user = new User(['role' => 'user']);

        $this->assertFalse($user->isAdmin());
    }

    public function test_premium_access_for_admin(): void
    {
        $user = new User(['role' => 'admin']);

        $this->assertTrue($user->canAccessPremiumContent());
    }

    public function test_premium_access_for_premium_subscriber(): void
    {
        $user = new User(['subscription_tier' => 'premium']);

        $this->assertTrue($user->canAccessPremiumContent());
    }

    public function test_no_premium_access_for_basic_user(): void
    {
        $user = new User([
            'role' => 'user',
            'subscription_tier' => 'basic',
        ]);

        $this->assertFalse($user->canAccessPremiumContent());
    }
}
```

## Testing Service Classes

Service classes with business logic are perfect for unit testing:

```php
// app/Services/PricingCalculator.php
class PricingCalculator
{
    public function __construct(
        private readonly float $taxRate = 0.10
    ) {}

    public function calculateSubtotal(array $items): float
    {
        return array_sum(array_map(
            fn ($item) => $item['price'] * $item['quantity'],
            $items
        ));
    }

    public function calculateTax(float $subtotal): float
    {
        return round($subtotal * $this->taxRate, 2);
    }

    public function calculateTotal(array $items): float
    {
        $subtotal = $this->calculateSubtotal($items);
        $tax = $this->calculateTax($subtotal);
        return $subtotal + $tax;
    }

    public function applyDiscount(float $amount, float $discountPercent): float
    {
        if ($discountPercent < 0 || $discountPercent > 100) {
            throw new \InvalidArgumentException('Discount must be between 0 and 100');
        }
        return round($amount * (1 - $discountPercent / 100), 2);
    }
}
```

```php
<?php

namespace Tests\Unit;

use App\Services\PricingCalculator;
use PHPUnit\Framework\TestCase;

class PricingCalculatorTest extends TestCase
{
    private PricingCalculator $calculator;

    protected function setUp(): void
    {
        parent::setUp();
        $this->calculator = new PricingCalculator(taxRate: 0.10);
    }

    public function test_calculates_subtotal(): void
    {
        $items = [
            ['price' => 10.00, 'quantity' => 2],
            ['price' => 5.00, 'quantity' => 3],
        ];

        $subtotal = $this->calculator->calculateSubtotal($items);

        $this->assertEquals(35.00, $subtotal);
    }

    public function test_calculates_tax(): void
    {
        $tax = $this->calculator->calculateTax(100.00);

        $this->assertEquals(10.00, $tax);
    }

    public function test_calculates_total_with_tax(): void
    {
        $items = [
            ['price' => 100.00, 'quantity' => 1],
        ];

        $total = $this->calculator->calculateTotal($items);

        $this->assertEquals(110.00, $total);
    }

    public function test_applies_discount(): void
    {
        $discounted = $this->calculator->applyDiscount(100.00, 20);

        $this->assertEquals(80.00, $discounted);
    }

    public function test_discount_throws_for_negative_percent(): void
    {
        $this->expectException(\InvalidArgumentException::class);
        $this->expectExceptionMessage('Discount must be between 0 and 100');

        $this->calculator->applyDiscount(100.00, -10);
    }

    public function test_discount_throws_for_over_100_percent(): void
    {
        $this->expectException(\InvalidArgumentException::class);

        $this->calculator->applyDiscount(100.00, 150);
    }

    public function test_empty_cart_returns_zero(): void
    {
        $subtotal = $this->calculator->calculateSubtotal([]);

        $this->assertEquals(0.00, $subtotal);
    }
}
```

## Testing Helper Functions

```php
// app/Helpers/StringHelper.php
function slugify(string $text): string
{
    $text = preg_replace('~[^\pL\d]+~u', '-', $text);
    $text = trim($text, '-');
    $text = strtolower($text);
    return preg_replace('~-+~', '-', $text);
}

function excerpt(string $text, int $length = 100): string
{
    if (strlen($text) <= $length) {
        return $text;
    }
    return substr($text, 0, $length) . '...';
}
```

```php
<?php

namespace Tests\Unit;

use PHPUnit\Framework\TestCase;

class StringHelperTest extends TestCase
{
    public function test_slugify_converts_spaces_to_hyphens(): void
    {
        $this->assertEquals('hello-world', slugify('Hello World'));
    }

    public function test_slugify_removes_special_characters(): void
    {
        $this->assertEquals('hello-world', slugify('Hello! World?'));
    }

    public function test_slugify_handles_unicode(): void
    {
        $this->assertEquals('cafe', slugify('Café'));
    }

    public function test_excerpt_returns_full_text_if_short(): void
    {
        $text = 'Short text';
        $this->assertEquals('Short text', excerpt($text, 100));
    }

    public function test_excerpt_truncates_long_text(): void
    {
        $text = str_repeat('a', 150);
        $result = excerpt($text, 100);

        $this->assertEquals(103, strlen($result)); // 100 + '...'
        $this->assertStringEndsWith('...', $result);
    }
}
```

## Resources

- [PHPUnit Documentation](https://phpunit.de/documentation.html) — Official PHPUnit documentation

---

> 📘 *This lesson is part of the [Laravel Testing Mastery](https://stanza.dev/courses/laravel-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*