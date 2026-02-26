---
source_course: "php-ddd"
source_lesson: "php-ddd-cqrs-pattern"
---

# CQRS: Command Query Responsibility Segregation

## Introduction

**CQRS** separates read operations (Queries) from write operations (Commands), allowing different models optimized for each purpose.

## Key Concepts

- **CQRS**
- **Flexibility**
- **Performance**
- **Scalability**
- **Simplicity**

## Real World Context

In production PHP applications, cqrs: command query separation helps teams build maintainable software by providing clear patterns for organizing complex business logic.

## Deep Dive

### Why CQRS?
```
Traditional: Same model for reads and writes
┌─────────────────────────────────────┐
│           Order Model                │
│  - Complex domain logic              │
│  - Used for writes AND reads         │
│  - Queries require loading aggregates│
└─────────────────────────────────────┘

CQRS: Separate models
┌─────────────────────┐  ┌─────────────────────┐
│   Command Model      │  │    Query Model      │
│  (Write Operations)  │  │  (Read Operations)  │
│                      │  │                     │
│  - Rich domain logic │  │  - Simple DTOs      │
│  - Aggregates        │  │  - Optimized views  │
│  - Consistency rules │  │  - Denormalized     │
└─────────────────────┘  └─────────────────────┘
```

### Commands
```php
<?php
namespace Application\Order\Command;

// Command: Intent to change state
final readonly class PlaceOrderCommand {
    public function __construct(
        public string $customerId,
        public array $items,
        public array $shippingAddress
    ) {}
}

final readonly class CancelOrderCommand {
    public function __construct(
        public string $orderId,
        public string $reason
    ) {}
}

// Command Handler
final class PlaceOrderHandler {
    public function __construct(
        private OrderRepository $orders,
        private CustomerRepository $customers,
        private EventDispatcher $events
    ) {}
    
    public function __invoke(PlaceOrderCommand $command): string {
        $customerId = CustomerId::fromString($command->customerId);
        $customer = $this->customers->findOrFail($customerId);
        
        $order = Order::place(
            $this->orders->nextIdentity(),
            $customerId,
            $this->buildItems($command->items)
        );
        
        $this->orders->save($order);
        $this->events->dispatchAll($order->pullDomainEvents());
        
        return $order->id()->toString();
    }
}
```

### Queries
```php
<?php
namespace Application\Order\Query;

// Query: Request for data
final readonly class GetOrderQuery {
    public function __construct(
        public string $orderId
    ) {}
}

final readonly class ListCustomerOrdersQuery {
    public function __construct(
        public string $customerId,
        public int $page = 1,
        public int $perPage = 20
    ) {}
}

// Query Handler - can bypass domain model for reads
final class GetOrderHandler {
    public function __construct(
        private \PDO $pdo  // Direct DB access for queries
    ) {}
    
    public function __invoke(GetOrderQuery $query): ?OrderView {
        $stmt = $this->pdo->prepare('
            SELECT 
                o.id,
                o.status,
                o.total_cents,
                o.currency,
                c.name as customer_name,
                o.created_at
            FROM orders o
            JOIN customers c ON c.id = o.customer_id
            WHERE o.id = ?
        ');
        $stmt->execute([$query->orderId]);
        $data = $stmt->fetch(\PDO::FETCH_ASSOC);
        
        if (!$data) {
            return null;
        }
        
        return new OrderView(
            id: $data['id'],
            status: $data['status'],
            total: number_format($data['total_cents'] / 100, 2),
            currency: $data['currency'],
            customerName: $data['customer_name'],
            createdAt: $data['created_at'],
            lines: $this->loadOrderLines($query->orderId)
        );
    }
}

// Read model - optimized for display
final readonly class OrderView {
    public function __construct(
        public string $id,
        public string $status,
        public string $total,
        public string $currency,
        public string $customerName,
        public string $createdAt,
        public array $lines
    ) {}
}
```

### Command/Query Bus
```php
<?php
namespace Infrastructure\Bus;

interface CommandBus {
    public function dispatch(object $command): mixed;
}

interface QueryBus {
    public function ask(object $query): mixed;
}

final class SimpleCommandBus implements CommandBus {
    /** @var array<class-string, callable> */
    private array $handlers = [];
    
    public function register(string $commandClass, callable $handler): void {
        $this->handlers[$commandClass] = $handler;
    }
    
    public function dispatch(object $command): mixed {
        $class = get_class($command);
        
        if (!isset($this->handlers[$class])) {
            throw new HandlerNotFound($class);
        }
        
        return ($this->handlers[$class])($command);
    }
}

// Usage in controller
final class OrderController {
    public function __construct(
        private CommandBus $commandBus,
        private QueryBus $queryBus
    ) {}
    
    public function place(Request $request): Response {
        $orderId = $this->commandBus->dispatch(
            new PlaceOrderCommand(
                customerId: $request->get('customer_id'),
                items: $request->get('items'),
                shippingAddress: $request->get('shipping_address')
            )
        );
        
        return new JsonResponse(['order_id' => $orderId], 201);
    }
    
    public function show(string $id): Response {
        $order = $this->queryBus->ask(new GetOrderQuery($id));
        
        if (!$order) {
            throw new NotFoundHttpException();
        }
        
        return new JsonResponse($order);
    }
}
```

### Benefits of CQRS
| Benefit | Description |
|---------|-------------|
| **Performance** | Queries optimized for reads, no aggregate loading |
| **Scalability** | Read and write sides scale independently |
| **Simplicity** | Each model focused on one responsibility |
| **Flexibility** | Different storage for reads vs writes |

## Common Pitfalls

1. **Overcomplicating simple cases** - Not every part of the application needs cqrs: command query separation. Apply it where complexity warrants the investment.
2. **Ignoring the ubiquitous language** - Naming classes and methods without input from domain experts leads to a model that does not reflect the business.
3. **Mixing infrastructure concerns** - Allowing framework dependencies to leak into the domain layer undermines the architectural benefits.

## Best Practices

1. **Start from the domain** - Model the business concepts first, then figure out persistence and infrastructure.
2. **Keep it simple** - Use the simplest pattern that solves the problem. Introduce complexity only when needed.
3. **Collaborate with domain experts** - The model should be shaped by business knowledge, not just technical preferences.

## Summary

- CQRS: Command Query Separation is a fundamental concept in Domain-Driven Design
- Proper implementation leads to more maintainable and expressive code
- Always align your implementation with the ubiquitous language
- Apply these patterns where business complexity justifies the investment

## Resources

- [CQRS Pattern](https://martinfowler.com/bliki/CQRS.html) — Martin Fowler on CQRS

---

> 📘 *This lesson is part of the [Domain-Driven Design with PHP](https://stanza.dev/courses/php-ddd) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*