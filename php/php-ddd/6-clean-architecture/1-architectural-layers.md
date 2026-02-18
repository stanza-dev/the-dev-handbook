---
source_course: "php-ddd"
source_lesson: "php-ddd-architectural-layers"
---

# Architectural Layers in DDD

**Clean Architecture** (also called Onion or Hexagonal Architecture) organizes code into concentric layers with strict dependency rules.

## The Dependency Rule

> Dependencies can only point **inward**. Inner layers know nothing about outer layers.

```
┌─────────────────────────────────────────────────────────────┐
│                    Infrastructure Layer                      │
│  (Frameworks, Database, External Services, UI)               │
│  ┌─────────────────────────────────────────────────────┐    │
│  │                Application Layer                     │    │
│  │  (Use Cases, Application Services, DTOs)             │    │
│  │  ┌─────────────────────────────────────────────┐    │    │
│  │  │              Domain Layer                   │    │    │
│  │  │  (Entities, Value Objects, Domain Services) │    │    │
│  │  │  ┌─────────────────────────────────────┐   │    │    │
│  │  │  │       Domain Model Core             │   │    │    │
│  │  │  │  (Business Rules, Invariants)       │   │    │    │
│  │  │  └─────────────────────────────────────┘   │    │    │
│  │  └─────────────────────────────────────────────┘    │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘

Dependencies: Infrastructure → Application → Domain
```

## Layer Responsibilities

### Domain Layer (Innermost)

```php
<?php
namespace Domain\Order;

// Pure business logic - no framework dependencies
final class Order {
    public function place(...): void { /* business rules */ }
    public function cancel(...): void { /* business rules */ }
}

final readonly class Money {
    public function add(Money $other): self { /* pure calculation */ }
}

// Interface defined in domain, implemented elsewhere
interface OrderRepository {
    public function save(Order $order): void;
}
```

### Application Layer

```php
<?php
namespace Application\Order;

// Use case orchestration
final class PlaceOrderHandler {
    public function __construct(
        private OrderRepository $orders,  // Domain interface
        private EventDispatcher $events   // Application interface
    ) {}
    
    public function __invoke(PlaceOrderCommand $command): string {
        // Orchestrate domain objects
        $order = Order::place(...);
        $this->orders->save($order);
        $this->events->dispatch($order->pullDomainEvents());
        return $order->id()->toString();
    }
}
```

### Infrastructure Layer (Outermost)

```php
<?php
namespace Infrastructure\Persistence;

use Domain\Order\OrderRepository;
use Domain\Order\Order;

// Implements domain interface
final class DoctrineOrderRepository implements OrderRepository {
    public function __construct(
        private EntityManagerInterface $em  // Framework dependency OK here
    ) {}
    
    public function save(Order $order): void {
        $this->em->persist($order);
        $this->em->flush();
    }
}

namespace Infrastructure\Http;

use Symfony\Component\HttpFoundation\Request;  // Framework OK here

final class OrderController {
    public function place(Request $request): Response {
        // Convert HTTP to application layer
        $command = new PlaceOrderCommand(...);
        $orderId = $this->commandBus->dispatch($command);
        return new JsonResponse(['id' => $orderId]);
    }
}
```

## Directory Structure

```
src/
├── Domain/                          # Core business logic
│   ├── Order/
│   │   ├── Order.php                # Aggregate root
│   │   ├── OrderLine.php            # Entity
│   │   ├── OrderId.php              # Value object
│   │   ├── OrderStatus.php          # Enum
│   │   ├── OrderRepository.php      # Interface
│   │   └── Events/
│   │       ├── OrderPlaced.php
│   │       └── OrderCancelled.php
│   ├── Customer/
│   └── Shared/
│       ├── Money.php
│       └── DomainEvent.php
│
├── Application/                     # Use cases
│   ├── Order/
│   │   ├── Command/
│   │   │   ├── PlaceOrderCommand.php
│   │   │   └── PlaceOrderHandler.php
│   │   ├── Query/
│   │   │   ├── GetOrderQuery.php
│   │   │   └── GetOrderHandler.php
│   │   └── DTO/
│   │       └── OrderResponse.php
│   └── Shared/
│       ├── CommandBus.php
│       └── QueryBus.php
│
└── Infrastructure/                  # External concerns
    ├── Persistence/
    │   ├── Doctrine/
    │   │   ├── DoctrineOrderRepository.php
    │   │   └── Mapping/
    │   └── InMemory/
    │       └── InMemoryOrderRepository.php
    ├── Http/
    │   ├── Controller/
    │   │   └── OrderController.php
    │   └── Middleware/
    ├── Event/
    │   ├── SymfonyEventDispatcher.php
    │   └── Listeners/
    └── External/
        ├── PaymentGateway/
        └── EmailService/
```

## Dependency Injection

```php
<?php
// services.yaml (Symfony) or similar DI configuration

// Domain interfaces bound to infrastructure implementations
return [
    // Repository bindings
    Domain\Order\OrderRepository::class => 
        Infrastructure\Persistence\Doctrine\DoctrineOrderRepository::class,
    
    Domain\Customer\CustomerRepository::class => 
        Infrastructure\Persistence\Doctrine\DoctrineCustomerRepository::class,
    
    // Application interfaces
    Application\Shared\EventDispatcher::class => 
        Infrastructure\Event\SymfonyEventDispatcher::class,
    
    Application\Shared\CommandBus::class => 
        Infrastructure\Bus\TacticianCommandBus::class,
];
```

## Benefits

| Benefit | How It's Achieved |
|---------|-------------------|
| **Testability** | Domain can be tested without infrastructure |
| **Flexibility** | Swap implementations without changing domain |
| **Maintainability** | Clear boundaries prevent spaghetti code |
| **Framework Independence** | Domain doesn't depend on frameworks |

## Resources

- [Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html) — Robert C. Martin's Clean Architecture post

---

> 📘 *This lesson is part of the [Domain-Driven Design with PHP](https://stanza.dev/courses/php-ddd) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*