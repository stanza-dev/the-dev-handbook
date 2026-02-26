---
source_course: "php-ddd"
source_lesson: "php-ddd-testing-architecture"
---

# Testing in Clean Architecture

## Introduction

Clean Architecture enables effective testing at every layer by isolating dependencies through interfaces. The domain layer can be unit-tested with no infrastructure, application services tested with simple test doubles, and adapters integration-tested against real infrastructure.

## Key Concepts

- **Unit Testing** - testing domain logic in isolation without any infrastructure dependencies
- **Integration Testing** - verifying that adapters correctly interact with real infrastructure
- **Test Double** - an object that substitutes a real dependency in tests (stubs, mocks, fakes)
- **Ports as Seams** - interfaces (ports) that allow swapping real implementations for test doubles

## Real World Context

A team working on an e-commerce platform tests domain pricing rules with simple unit tests that run in milliseconds, validates repository implementations against a test database, and uses in-memory fakes for application service tests. This layered testing strategy provides fast feedback while still verifying integration points.

## Deep Dive

### Testing the Domain Layer

Domain entities and value objects have no dependencies on external systems, making them the easiest to test.

```php
<?php
declare(strict_types=1);

namespace Tests\Domain\Order;

use PHPUnit\Framework\TestCase;

final class OrderTest extends TestCase
{
    public function test_cannot_submit_empty_order(): void
    {
        $order = Order::create(
            OrderId::generate(),
            CustomerId::fromString('cust-1')
        );
        $order->setShippingAddress(ShippingAddress::fromArray([
            'street' => '123 Main St',
            'city' => 'Springfield',
            'postal_code' => '62701',
            'country' => 'US',
        ]));

        $this->expectException(OrderException::class);
        $order->submit();
    }

    public function test_submit_records_domain_event(): void
    {
        $order = $this->createOrderWithItems();
        $order->submit();

        $events = $order->pullDomainEvents();

        $this->assertCount(2, $events); // OrderCreated + OrderSubmitted
        $this->assertInstanceOf(OrderSubmitted::class, $events[1]);
    }

    public function test_cancel_changes_status(): void
    {
        $order = $this->createOrderWithItems();
        $order->submit();

        $order->cancel(new CancellationReason('Customer changed mind'));

        $this->assertEquals(OrderStatus::Cancelled, $order->status());
    }

    private function createOrderWithItems(): Order
    {
        $order = Order::create(
            OrderId::generate(),
            CustomerId::fromString('cust-1')
        );
        $order->addProduct(
            ProductId::fromString('prod-1'),
            new ProductName('Widget'),
            new Quantity(2),
            Money::fromCents(1500)
        );
        $order->setShippingAddress(ShippingAddress::fromArray([
            'street' => '123 Main St',
            'city' => 'Springfield',
            'postal_code' => '62701',
            'country' => 'US',
        ]));
        return $order;
    }
}
```

### Testing Value Objects

```php
<?php
final class MoneyTest extends TestCase
{
    public function test_addition(): void
    {
        $a = Money::fromCents(1000);
        $b = Money::fromCents(500);

        $result = $a->add($b);

        $this->assertEquals(1500, $result->cents());
        // Original values unchanged (immutability)
        $this->assertEquals(1000, $a->cents());
    }

    public function test_cannot_add_different_currencies(): void
    {
        $usd = Money::fromCents(1000, Currency::USD);
        $eur = Money::fromCents(500, Currency::EUR);

        $this->expectException(CurrencyMismatch::class);
        $usd->add($eur);
    }
}
```

### In-Memory Fakes for Application Layer Testing

```php
<?php
namespace Tests\Infrastructure\Fake;

final class InMemoryOrderRepository implements OrderRepository
{
    /** @var array<string, Order> */
    private array $orders = [];
    private int $nextId = 1;

    public function save(Order $order): void
    {
        $this->orders[$order->id()->toString()] = $order;
    }

    public function find(OrderId $id): ?Order
    {
        return $this->orders[$id->toString()] ?? null;
    }

    public function findOrFail(OrderId $id): Order
    {
        return $this->find($id)
            ?? throw OrderNotFound::withId($id);
    }

    public function nextIdentity(): OrderId
    {
        return OrderId::fromString('order-' . $this->nextId++);
    }

    // Test helpers
    public function count(): int { return count($this->orders); }
    public function clear(): void { $this->orders = []; }
}

final class CollectingEventDispatcher implements EventDispatcher
{
    /** @var object[] */
    private array $dispatched = [];

    public function dispatch(object $event): void
    {
        $this->dispatched[] = $event;
    }

    public function dispatchAll(array $events): void
    {
        foreach ($events as $event) {
            $this->dispatch($event);
        }
    }

    /** @return object[] */
    public function dispatched(): array { return $this->dispatched; }
    public function reset(): void { $this->dispatched = []; }
}
```

### Testing Application Services

```php
<?php
final class OrderApplicationServiceTest extends TestCase
{
    private InMemoryOrderRepository $orders;
    private CollectingEventDispatcher $events;
    private OrderApplicationService $service;

    protected function setUp(): void
    {
        $this->orders = new InMemoryOrderRepository();
        $this->events = new CollectingEventDispatcher();

        $this->service = new OrderApplicationService(
            orders: $this->orders,
            customers: new InMemoryCustomerRepository(),
            catalog: new StubProductCatalog(),
            inventory: new AlwaysAvailableInventory(),
            payments: new AlwaysSuccessfulPayments(),
            events: $this->events,
            tx: new NoOpTransactionManager(),
        );
    }

    public function test_create_order_persists_and_dispatches_events(): void
    {
        $command = new CreateOrderCommand(
            customerId: 'cust-1',
            items: [new OrderItemInput('prod-1', 2)],
        );

        $orderId = $this->service->createOrder($command);

        $this->assertNotNull($this->orders->find($orderId));
        $this->assertCount(1, $this->events->dispatched());
    }

    public function test_submit_order_checks_inventory(): void
    {
        $inventory = new NeverAvailableInventory();
        $service = new OrderApplicationService(
            orders: $this->orders,
            inventory: $inventory,
            // ... other dependencies
        );

        $orderId = $this->createSampleOrder();

        $this->expectException(InsufficientInventory::class);
        $service->submitOrder(new SubmitOrderCommand($orderId->toString()));
    }
}
```

### Integration Testing Adapters

```php
<?php
final class DoctrineOrderRepositoryTest extends TestCase
{
    private EntityManagerInterface $em;
    private DoctrineOrderRepository $repository;

    protected function setUp(): void
    {
        $this->em = TestEntityManagerFactory::create();
        $this->repository = new DoctrineOrderRepository($this->em);
    }

    public function test_save_and_find_order(): void
    {
        $order = Order::create(
            OrderId::fromString('order-1'),
            CustomerId::fromString('cust-1')
        );
        $order->addProduct(
            ProductId::fromString('prod-1'),
            new ProductName('Widget'),
            new Quantity(1),
            Money::fromCents(1000)
        );

        $this->repository->save($order);
        $this->em->clear();

        $found = $this->repository->find(OrderId::fromString('order-1'));

        $this->assertNotNull($found);
        $this->assertEquals(1000, $found->total()->cents());
        $this->assertCount(1, $found->lines());
    }

    protected function tearDown(): void
    {
        $this->em->getConnection()->executeStatement('DELETE FROM order_lines');
        $this->em->getConnection()->executeStatement('DELETE FROM orders');
    }
}
```

## Common Pitfalls

1. **Mocking domain objects** - Domain entities and value objects should be used directly in tests, not mocked. Mocking them hides real behavior and can lead to tests that pass despite broken domain logic.
2. **Testing implementation details** - Focus tests on behavior and outcomes, not internal method calls. Tests coupled to implementation break during refactoring even when behavior is correct.
3. **Skipping integration tests** - Unit tests with fakes verify logic but not actual infrastructure interaction. Always integration-test your adapters against real databases and services.

## Best Practices

1. **Use the testing pyramid** - Many fast domain unit tests, fewer application service tests with fakes, and a small number of slow integration tests for adapters.
2. **Make test doubles reusable** - Build a library of in-memory fakes and stubs that can be shared across all application service tests.
3. **Test domain events explicitly** - Verify that domain operations record the correct events, since downstream behavior often depends on them.

## Summary

- Clean Architecture makes each layer independently testable through clear interfaces
- Domain objects are tested directly with plain unit tests and no mocking
- In-memory fakes substitute infrastructure dependencies in application service tests
- Integration tests verify adapters against real databases and external services
- A layered testing strategy provides both fast feedback and thorough coverage

## Resources

- [Testing Strategies in a Microservice Architecture](https://martinfowler.com/articles/microservice-testing/) — Martin Fowler's comprehensive guide to testing strategies

---

> 📘 *This lesson is part of the [Domain-Driven Design with PHP](https://stanza.dev/courses/php-ddd) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*