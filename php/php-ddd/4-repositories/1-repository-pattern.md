---
source_course: "php-ddd"
source_lesson: "php-ddd-repository-pattern"
---

# The Repository Pattern

## Introduction

A **Repository** mediates between the domain and data mapping layers. It provides a collection-like interface for accessing domain objects while hiding persistence details.

## Key Concepts

- **Repository**

## Real World Context

In production PHP applications, the repository pattern helps teams build maintainable software by providing clear patterns for organizing complex business logic.

## Deep Dive

### Why Repositories?
```php
<?php
// BAD: Domain coupled to persistence
class OrderService {
    public function getOrder(string $id): Order {
        $pdo = new PDO(...);
        $stmt = $pdo->prepare('SELECT * FROM orders WHERE id = ?');
        $stmt->execute([$id]);
        $data = $stmt->fetch();
        return new Order($data['id'], ...);
    }
}

// GOOD: Domain uses abstraction
class OrderService {
    public function __construct(
        private OrderRepository $orders  // Interface, not implementation
    ) {}
    
    public function getOrder(OrderId $id): Order {
        return $this->orders->find($id);
    }
}
```

### Repository Interface (Domain Layer)
```php
<?php
namespace Domain\Ordering;

interface OrderRepository {
    public function find(OrderId $id): ?Order;
    
    public function findOrFail(OrderId $id): Order;
    
    public function save(Order $order): void;
    
    public function remove(Order $order): void;
    
    public function nextIdentity(): OrderId;
    
    /**
     * @return Order[]
     */
    public function findByCustomer(CustomerId $customerId): array;
    
    /**
     * @return Order[]
     */
    public function findPendingOlderThan(\DateTimeImmutable $date): array;
}
```

### Repository Implementation (Infrastructure Layer)
```php
<?php
namespace Infrastructure\Persistence\Doctrine;

use Doctrine\ORM\EntityManagerInterface;
use Domain\Ordering\Order;
use Domain\Ordering\OrderId;
use Domain\Ordering\OrderRepository;

final class DoctrineOrderRepository implements OrderRepository {
    public function __construct(
        private EntityManagerInterface $em
    ) {}
    
    public function find(OrderId $id): ?Order {
        return $this->em->find(Order::class, $id->toString());
    }
    
    public function findOrFail(OrderId $id): Order {
        $order = $this->find($id);
        
        if ($order === null) {
            throw OrderNotFound::withId($id);
        }
        
        return $order;
    }
    
    public function save(Order $order): void {
        $this->em->persist($order);
        $this->em->flush();
    }
    
    public function remove(Order $order): void {
        $this->em->remove($order);
        $this->em->flush();
    }
    
    public function nextIdentity(): OrderId {
        return OrderId::generate();
    }
    
    public function findByCustomer(CustomerId $customerId): array {
        return $this->em->createQueryBuilder()
            ->select('o')
            ->from(Order::class, 'o')
            ->where('o.customerId = :customerId')
            ->setParameter('customerId', $customerId->toString())
            ->orderBy('o.createdAt', 'DESC')
            ->getQuery()
            ->getResult();
    }
    
    public function findPendingOlderThan(\DateTimeImmutable $date): array {
        return $this->em->createQueryBuilder()
            ->select('o')
            ->from(Order::class, 'o')
            ->where('o.status = :status')
            ->andWhere('o.createdAt < :date')
            ->setParameter('status', OrderStatus::Pending->value)
            ->setParameter('date', $date)
            ->getQuery()
            ->getResult();
    }
}
```

### In-Memory Repository for Testing
```php
<?php
namespace Infrastructure\Persistence\InMemory;

final class InMemoryOrderRepository implements OrderRepository {
    /** @var array<string, Order> */
    private array $orders = [];
    
    public function find(OrderId $id): ?Order {
        return $this->orders[$id->toString()] ?? null;
    }
    
    public function findOrFail(OrderId $id): Order {
        return $this->find($id)
            ?? throw OrderNotFound::withId($id);
    }
    
    public function save(Order $order): void {
        $this->orders[$order->id()->toString()] = $order;
    }
    
    public function remove(Order $order): void {
        unset($this->orders[$order->id()->toString()]);
    }
    
    public function nextIdentity(): OrderId {
        return OrderId::generate();
    }
    
    public function findByCustomer(CustomerId $customerId): array {
        return array_filter(
            $this->orders,
            fn(Order $o) => $o->customerId()->equals($customerId)
        );
    }
    
    // Helper for tests
    public function clear(): void {
        $this->orders = [];
    }
    
    public function count(): int {
        return count($this->orders);
    }
}
```

### Repository Guidelines
### 1. One Repository Per Aggregate

```php
<?php
// GOOD: Repository for aggregate root only
interface OrderRepository {
    public function save(Order $order): void;
}

// BAD: Repository for internal entity
interface OrderLineRepository {  // OrderLine is inside Order aggregate!
    public function save(OrderLine $line): void;
}
```

### 2. Return Domain Objects, Not Arrays

```php
<?php
// BAD
interface OrderRepository {
    public function find(string $id): ?array;  // Raw data
}

// GOOD
interface OrderRepository {
    public function find(OrderId $id): ?Order;  // Domain object
}
```

### 3. Domain-Focused Query Methods

```php
<?php
// BAD: Technical query methods
interface OrderRepository {
    public function findWhere(array $criteria): array;
    public function findBySql(string $sql): array;
}

// GOOD: Business-meaningful methods
interface OrderRepository {
    public function findPendingForCustomer(CustomerId $id): array;
    public function findRequiringAttention(): array;
    public function findRecentlyCompleted(int $days): array;
}
```

## Common Pitfalls

1. **Overcomplicating simple cases** - Not every part of the application needs the repository pattern. Apply it where complexity warrants the investment.
2. **Ignoring the ubiquitous language** - Naming classes and methods without input from domain experts leads to a model that does not reflect the business.
3. **Mixing infrastructure concerns** - Allowing framework dependencies to leak into the domain layer undermines the architectural benefits.

## Best Practices

1. **Start from the domain** - Model the business concepts first, then figure out persistence and infrastructure.
2. **Keep it simple** - Use the simplest pattern that solves the problem. Introduce complexity only when needed.
3. **Collaborate with domain experts** - The model should be shaped by business knowledge, not just technical preferences.

## Summary

- The Repository Pattern is a fundamental concept in Domain-Driven Design
- Proper implementation leads to more maintainable and expressive code
- Always align your implementation with the ubiquitous language
- Apply these patterns where business complexity justifies the investment

## Resources

- [Repository Pattern](https://martinfowler.com/eaaCatalog/repository.html) — Martin Fowler's catalog entry on Repository

---

> 📘 *This lesson is part of the [Domain-Driven Design with PHP](https://stanza.dev/courses/php-ddd) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*