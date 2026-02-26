---
source_course: "php-ddd"
source_lesson: "php-ddd-persistence-strategies"
---

# Persistence Strategies

## Introduction

How you persist domain objects significantly impacts your architecture. Let's explore different approaches.

## Key Concepts

- **Active Record** - a pattern where entities know how to persist themselves
- **Data Mapper** - a pattern that separates domain objects from persistence logic
- **Event Sourcing** - storing events instead of current state
- **Manual Mapping** - hand-writing the translation between domain and database
- **Doctrine ORM** - a PHP Data Mapper implementation for database abstraction

## Real World Context

In production PHP applications, persistence strategies helps teams build maintainable software by providing clear patterns for organizing complex business logic.

## Deep Dive

### Active Record vs Data Mapper
### Active Record

The entity knows how to persist itself:

```php
<?php
// Active Record - entity coupled to persistence
class Order extends Model {  // Laravel Eloquent style
    public function save(): void {
        $this->connection->update(...);
    }
}

$order->total = 100;
$order->save();
```

❌ **Problem**: Domain mixed with infrastructure

### Data Mapper (DDD Preferred)

Separate classes handle persistence:

```php
<?php
// Data Mapper - domain stays pure
final class Order {  // Pure domain object
    // No persistence code here
}

class OrderRepository {
    public function save(Order $order): void {
        // Persistence handled separately
    }
}
```

✅ **Benefit**: Clean domain model

### Manual Mapping
```php
<?php
final class PdoOrderRepository implements OrderRepository {
    public function __construct(
        private \PDO $pdo
    ) {}
    
    public function find(OrderId $id): ?Order {
        $stmt = $this->pdo->prepare('
            SELECT o.*, GROUP_CONCAT(l.id) as line_ids
            FROM orders o
            LEFT JOIN order_lines l ON l.order_id = o.id
            WHERE o.id = ?
            GROUP BY o.id
        ');
        $stmt->execute([$id->toString()]);
        $data = $stmt->fetch(\PDO::FETCH_ASSOC);
        
        if ($data === false) {
            return null;
        }
        
        return $this->hydrate($data);
    }
    
    public function save(Order $order): void {
        $this->pdo->beginTransaction();
        
        try {
            // Save order
            $this->pdo->prepare('
                INSERT INTO orders (id, customer_id, status, total_cents, created_at)
                VALUES (?, ?, ?, ?, ?)
                ON DUPLICATE KEY UPDATE
                    status = VALUES(status),
                    total_cents = VALUES(total_cents)
            ')->execute([
                $order->id()->toString(),
                $order->customerId()->toString(),
                $order->status()->value,
                $order->total()->cents(),
                $order->createdAt()->format('Y-m-d H:i:s')
            ]);
            
            // Save lines
            $this->saveLines($order);
            
            $this->pdo->commit();
        } catch (\Exception $e) {
            $this->pdo->rollBack();
            throw $e;
        }
    }
    
    private function hydrate(array $data): Order {
        $lines = $this->loadLines($data['id']);
        
        return Order::reconstitute(
            id: OrderId::fromString($data['id']),
            customerId: CustomerId::fromString($data['customer_id']),
            status: OrderStatus::from($data['status']),
            total: Money::fromCents((int)$data['total_cents']),
            lines: $lines,
            createdAt: new \DateTimeImmutable($data['created_at'])
        );
    }
}
```

### Using Doctrine ORM
```php
<?php
// Domain entity with Doctrine attributes
namespace Domain\Ordering;

use Doctrine\ORM\Mapping as ORM;

#[ORM\Entity]
#[ORM\Table(name: 'orders')]
final class Order {
    #[ORM\Id]
    #[ORM\Column(type: 'string', length: 36)]
    private string $id;
    
    #[ORM\Column(type: 'string', length: 36)]
    private string $customerId;
    
    #[ORM\Column(type: 'string', enumType: OrderStatus::class)]
    private OrderStatus $status;
    
    #[ORM\Embedded(class: Money::class)]
    private Money $total;
    
    #[ORM\OneToMany(targetEntity: OrderLine::class, mappedBy: 'order', cascade: ['persist', 'remove'])]
    private Collection $lines;
    
    // Domain methods...
}

// Value Object mapping
#[ORM\Embeddable]
final class Money {
    #[ORM\Column(type: 'integer')]
    private int $cents;
    
    #[ORM\Column(type: 'string', length: 3)]
    private string $currency;
}
```

### Event Sourcing (Advanced)
Instead of storing current state, store all events:

```php
<?php
final class EventSourcedOrderRepository implements OrderRepository {
    public function __construct(
        private EventStore $eventStore
    ) {}
    
    public function find(OrderId $id): ?Order {
        $events = $this->eventStore->getEventsFor($id->toString());
        
        if (empty($events)) {
            return null;
        }
        
        // Rebuild order from events
        return Order::fromHistory($events);
    }
    
    public function save(Order $order): void {
        $events = $order->pullDomainEvents();
        $this->eventStore->append($order->id()->toString(), $events);
    }
}

// Event Store
final class SqlEventStore implements EventStore {
    public function append(string $aggregateId, array $events): void {
        foreach ($events as $event) {
            $this->pdo->prepare('
                INSERT INTO events (aggregate_id, event_type, payload, occurred_at)
                VALUES (?, ?, ?, ?)
            ')->execute([
                $aggregateId,
                get_class($event),
                json_encode($event),
                $event->occurredAt()->format('Y-m-d H:i:s.u')
            ]);
        }
    }
}
```

## Common Pitfalls

1. **Overcomplicating simple cases** - Not every part of the application needs persistence strategies. Apply it where complexity warrants the investment.
2. **Ignoring the ubiquitous language** - Naming classes and methods without input from domain experts leads to a model that does not reflect the business.
3. **Mixing infrastructure concerns** - Allowing framework dependencies to leak into the domain layer undermines the architectural benefits.

## Best Practices

1. **Start from the domain** - Model the business concepts first, then figure out persistence and infrastructure.
2. **Keep it simple** - Use the simplest pattern that solves the problem. Introduce complexity only when needed.
3. **Collaborate with domain experts** - The model should be shaped by business knowledge, not just technical preferences.

## Summary

- Persistence Strategies is a fundamental concept in Domain-Driven Design
- Proper implementation leads to more maintainable and expressive code
- Always align your implementation with the ubiquitous language
- Apply these patterns where business complexity justifies the investment

## Resources

- [Doctrine ORM Documentation](https://www.doctrine-project.org/projects/doctrine-orm/en/current/index.html) — Official Doctrine ORM documentation

---

> 📘 *This lesson is part of the [Domain-Driven Design with PHP](https://stanza.dev/courses/php-ddd) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*