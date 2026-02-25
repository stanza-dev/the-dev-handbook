---
source_course: "php-testing"
source_lesson: "php-testing-fixtures"
---

# Fixtures and Test Data Management

## Introduction
As your test suite grows, managing test data becomes a challenge. Duplicated setup code, inconsistent data, and brittle fixtures slow down development and make tests hard to maintain. Fixture patterns — factories, builders, seeders, and shared fixtures — bring structure and reusability to your test data.

## Key Concepts
- **Factory Pattern**: A class that creates test objects with sensible defaults, allowing individual tests to override only the fields they care about.
- **Builder Pattern**: A fluent interface for constructing complex test objects step by step, improving readability for objects with many optional fields.
- **Shared Fixtures**: Common data setup shared across multiple tests via `setUp()` or `setUpBeforeClass()`, reducing duplication.
- **Database Seeder**: A script that populates the database with a baseline dataset used by multiple integration tests.

## Real World Context
Without fixture patterns, every test that needs a `User` object repeats the same creation code with all required fields. When a new required field is added to `User`, every test breaks. Factories centralize object creation so you update the default in one place and all tests continue to work.

## Deep Dive
The factory pattern is the most powerful fixture technique. A factory creates objects with reasonable defaults and lets each test override only what matters for that specific scenario.

Here is a simple factory implementation:

```php
<?php
declare(strict_types=1);

class UserFactory
{
    private static int $sequence = 0;

    public static function create(array $overrides = []): User
    {
        self::$sequence++;

        $defaults = [
            'id' => self::$sequence,
            'email' => "user" . self::$sequence . "@test.com",
            'name' => 'Test User ' . self::$sequence,
            'role' => 'member',
            'createdAt' => new \DateTimeImmutable('2025-01-01'),
        ];

        $data = array_merge($defaults, $overrides);

        return new User(
            id: $data['id'],
            email: $data['email'],
            name: $data['name'],
            role: $data['role'],
            createdAt: $data['createdAt'],
        );
    }

    public static function createAdmin(array $overrides = []): User
    {
        return self::create(array_merge(['role' => 'admin'], $overrides));
    }
}
```

The factory uses a sequence counter to generate unique emails and IDs. The `createAdmin()` convenience method shows how to create preset variants.

Using the factory in tests is clean and expressive:

```php
<?php
class PermissionServiceTest extends TestCase
{
    public function testAdminCanDeleteUsers(): void
    {
        $admin = UserFactory::createAdmin();
        $target = UserFactory::create();

        $service = new PermissionService();

        $this->assertTrue($service->canDelete($admin, $target));
    }

    public function testMemberCannotDeleteUsers(): void
    {
        $member = UserFactory::create(['role' => 'member']);
        $target = UserFactory::create();

        $service = new PermissionService();

        $this->assertFalse($service->canDelete($member, $target));
    }
}
```

Notice how the tests only specify the data that matters for the scenario. The admin test uses `createAdmin()`, and the member test overrides only the `role` field.

For objects with many optional fields, the builder pattern provides a fluent API:

```php
<?php
class OrderBuilder
{
    private string $customerEmail = 'customer@test.com';
    private string $status = 'pending';
    private array $items = [];
    private ?string $couponCode = null;
    private string $currency = 'USD';

    public static function anOrder(): self
    {
        return new self();
    }

    public function withCustomer(string $email): self
    {
        $clone = clone $this;
        $clone->customerEmail = $email;
        return $clone;
    }

    public function withStatus(string $status): self
    {
        $clone = clone $this;
        $clone->status = $status;
        return $clone;
    }

    public function withItem(string $product, float $price, int $qty = 1): self
    {
        $clone = clone $this;
        $clone->items[] = ['product' => $product, 'price' => $price, 'quantity' => $qty];
        return $clone;
    }

    public function withCoupon(string $code): self
    {
        $clone = clone $this;
        $clone->couponCode = $code;
        return $clone;
    }

    public function build(): Order
    {
        return new Order(
            customerEmail: $this->customerEmail,
            status: $this->status,
            items: $this->items,
            couponCode: $this->couponCode,
            currency: $this->currency,
        );
    }
}
```

The builder uses immutable `clone` to prevent shared state between tests. Each `with*()` method returns a new builder instance.

Using the builder reads like a specification:

```php
<?php
public function testCouponDiscountApplied(): void
{
    $order = OrderBuilder::anOrder()
        ->withItem('Laptop', 999.99)
        ->withItem('Mouse', 29.99)
        ->withCoupon('SAVE10')
        ->build();

    $calculator = new OrderCalculator($this->couponService);
    $total = $calculator->calculateTotal($order);

    // 10% off: (999.99 + 29.99) * 0.9 = 926.98
    $this->assertEqualsWithDelta(926.98, $total, 0.01);
}
```

The builder chain clearly communicates what kind of order is being tested.

For database integration tests, a seeder class populates baseline data:

```php
<?php
class TestDatabaseSeeder
{
    public function __construct(private readonly PDO $pdo) {}

    public function seedUsers(): array
    {
        $users = [
            ['alice@test.com', 'Alice', 'admin'],
            ['bob@test.com', 'Bob', 'member'],
            ['carol@test.com', 'Carol', 'member'],
        ];

        $stmt = $this->pdo->prepare(
            'INSERT INTO users (email, name, role) VALUES (?, ?, ?)'
        );

        $ids = [];
        foreach ($users as $user) {
            $stmt->execute($user);
            $ids[] = (int) $this->pdo->lastInsertId();
        }

        return $ids;
    }

    public function seedOrdersForUser(int $userId, int $count = 3): void
    {
        $stmt = $this->pdo->prepare(
            'INSERT INTO orders (user_id, total, status) VALUES (?, ?, ?)'
        );

        for ($i = 0; $i < $count; $i++) {
            $stmt->execute([$userId, ($i + 1) * 25.00, 'pending']);
        }
    }
}
```

The seeder provides reusable data setup that multiple test classes can call in their `setUp()` methods.

## Common Pitfalls
1. **Factories with stale defaults** — When you add a new required field to a domain object but forget to update the factory, all tests using that factory break with a cryptic constructor error. Always update factories when domain objects change.
2. **Over-seeding** — Inserting too much data in `setUp()` makes every test slow and makes it hard to tell which data is relevant to which test. Seed the minimum data needed and let individual tests add their specific requirements.

## Best Practices
1. **One factory per domain object** — Keep factories in a `tests/Factories/` directory alongside your test suite. Each factory handles one entity type and provides convenience methods for common variants (e.g., `createAdmin()`, `createExpiredSubscription()`).
2. **Builders for complex objects, factories for simple ones** — Use factories when objects have few fields and most tests use defaults. Switch to the builder pattern when objects have many optional fields and tests need fine-grained control over the setup.

## Summary
- Factories create test objects with sensible defaults, reducing duplication and centralizing object construction.
- The builder pattern provides a fluent API for constructing complex objects with many optional fields.
- Seeders populate databases with baseline data for integration tests.
- Always update factories when domain objects change to prevent cascading test failures.
- Keep test data minimal — each test should seed only the data it actually needs.

## Code Examples

**Product factory with presets for out-of-stock and inactive variants, used in cart service tests**

```php
<?php
declare(strict_types=1);

// Factory with sequence counter and convenient presets
class ProductFactory
{
    private static int $seq = 0;

    public static function create(array $overrides = []): Product
    {
        self::$seq++;
        $defaults = [
            'id' => self::$seq,
            'name' => 'Product ' . self::$seq,
            'sku' => 'SKU-' . str_pad((string) self::$seq, 5, '0', STR_PAD_LEFT),
            'price' => 9.99,
            'stock' => 100,
            'active' => true,
        ];

        $data = array_merge($defaults, $overrides);
        return new Product(...$data);
    }

    public static function createOutOfStock(array $overrides = []): Product
    {
        return self::create(array_merge(['stock' => 0], $overrides));
    }

    public static function createInactive(array $overrides = []): Product
    {
        return self::create(array_merge(['active' => false], $overrides));
    }
}

// Usage in tests
class CartServiceTest extends TestCase
{
    public function testCannotAddOutOfStockProduct(): void
    {
        $product = ProductFactory::createOutOfStock();
        $cart = new CartService();

        $this->expectException(OutOfStockException::class);
        $cart->addItem($product, quantity: 1);
    }

    public function testCanAddActiveProductWithStock(): void
    {
        $product = ProductFactory::create(['price' => 24.99]);
        $cart = new CartService();

        $cart->addItem($product, quantity: 2);

        $this->assertSame(49.98, $cart->getTotal());
        $this->assertCount(1, $cart->getItems());
    }
}
?>
```


## Resources

- [PHPUnit Fixtures Documentation](https://docs.phpunit.de/en/12.0/fixtures.html) — Official PHPUnit 12 documentation on shared fixtures and setUp/tearDown patterns
- [Object Mother and Test Data Builder Patterns](https://martinfowler.com/bliki/ObjectMother.html) — Martin Fowler's overview of the Object Mother pattern, closely related to test factories

---

> 📘 *This lesson is part of the [PHP Testing & Quality Assurance](https://stanza.dev/courses/php-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*