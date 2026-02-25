---
source_course: "php-databases"
source_lesson: "php-databases-active-record-tradeoffs"
---

# Active Record Trade-offs & Alternatives

## Introduction

Active Record is beloved for its simplicity but criticized for coupling domain logic to persistence. Understanding its strengths, weaknesses, and alternatives helps you choose the right pattern for each project's complexity level.

## Key Concepts

- **Active Record**: Objects manage their own persistence. Simple but couples domain logic to the database layer.
- **Data Mapper**: A separate mapper class handles persistence. Domain objects have no knowledge of the database.
- **Repository Pattern**: A collection-like interface that abstracts the underlying persistence mechanism.

## Real World Context

A startup building an MVP chooses Active Record for speed of development. A fintech company processing millions in transactions chooses Data Mapper to keep business rules testable independently of the database. Neither choice is wrong — they solve different problems at different scales.

## Deep Dive

Active Record's strengths:

```php
<?php
// Extremely concise CRUD
$user = User::find(1);
$user->name = 'New Name';
$user->save();

// Convention over configuration
// Class name -> table name, properties -> columns
// Minimal boilerplate for simple models
```

Active Record's weaknesses become apparent in complex domains:

```php
<?php
// Problem 1: Business logic mixed with persistence
class Order extends ActiveRecord {
    public function calculateTotal(): float {
        // This method knows about both business rules AND database
        $items = OrderItem::where('order_id', $this->id)->get();
        $subtotal = array_sum(array_column(
            array_map(fn($i) => $i->toArray(), $items), 'price'
        ));
        $tax = $subtotal * 0.1;
        $this->attributes['total'] = $subtotal + $tax;
        $this->save(); // Persistence mixed with calculation!
        return $this->total;
    }
}

// Problem 2: Hard to test without a database
// Every test needs a real database connection
// Mocking static methods like User::find() is difficult
```

The Data Mapper pattern separates concerns:

```php
<?php
// Domain object — no database knowledge
class Order {
    public function __construct(
        public readonly ?int $id,
        public readonly int $userId,
        public float $total = 0.0,
        public array $items = [],
    ) {}
    
    public function calculateTotal(): float {
        $subtotal = array_sum(array_map(
            fn(OrderItem $i) => $i->price * $i->quantity,
            $this->items
        ));
        $this->total = $subtotal * 1.1; // 10% tax
        return $this->total;
    }
}

// Mapper handles persistence separately
class OrderMapper {
    public function __construct(private PDO $pdo) {}
    
    public function findById(int $id): ?Order {
        $stmt = $this->pdo->prepare('SELECT * FROM orders WHERE id = :id');
        $stmt->execute(['id' => $id]);
        $row = $stmt->fetch();
        return $row ? $this->mapToEntity($row) : null;
    }
    
    public function save(Order $order): void {
        if ($order->id === null) {
            $this->insert($order);
        } else {
            $this->update($order);
        }
    }
    
    private function mapToEntity(array $row): Order {
        return new Order(
            id: (int) $row['id'],
            userId: (int) $row['user_id'],
            total: (float) $row['total'],
        );
    }
}
```

The Repository pattern provides a collection-like interface:

```php
<?php
interface OrderRepository {
    public function findById(int $id): ?Order;
    public function findByUser(int $userId): array;
    public function save(Order $order): void;
    public function delete(Order $order): void;
}

class PdoOrderRepository implements OrderRepository {
    public function __construct(private PDO $pdo) {}
    // Implementation using PDO...
}

class InMemoryOrderRepository implements OrderRepository {
    private array $orders = [];
    // Implementation for testing...
}
```

When to use which:

| Pattern | Best For | Avoid When |
|---------|----------|------------|
| Active Record | CRUD apps, MVPs, admin panels | Complex business rules |
| Data Mapper | Domain-rich applications | Simple CRUD |
| Repository | Testable domain logic | Over-engineering simple apps |

## Common Pitfalls

1. **Using Active Record for complex business domains** — When business logic requires testability independent of the database, Active Record's coupling becomes a liability.
2. **Using Data Mapper for simple CRUD** — If your application is mostly forms-over-data with little business logic, Data Mapper adds unnecessary abstraction layers.

## Best Practices

1. **Start with Active Record, migrate when needed** — Most applications start simple. Refactoring from Active Record to Data Mapper when complexity demands it is straightforward.
2. **Use Repository interfaces even with Active Record** — Wrapping Active Record calls behind a Repository interface lets you swap implementations later without changing business logic.

## Summary

- Active Record couples persistence with domain logic — ideal for CRUD and MVPs.
- Data Mapper separates domain objects from persistence — better for complex business logic and testability.
- Repository pattern abstracts persistence behind a collection-like interface.
- Start with Active Record and migrate to Data Mapper only when domain complexity demands it.

## Code Examples

**Repository pattern wrapping Active Record**

```php
<?php
declare(strict_types=1);

// Repository wrapping Active Record for future flexibility
interface UserRepository {
    public function findById(int $id): ?User;
    public function findActive(): array;
    public function save(User $user): bool;
}

class EloquentUserRepository implements UserRepository {
    public function findById(int $id): ?User {
        return User::find($id);
    }
    
    public function findActive(): array {
        return User::where('status', 'active')->get();
    }
    
    public function save(User $user): bool {
        return $user->save();
    }
}

// Controller depends on interface, not implementation
class UserController {
    public function __construct(private UserRepository $users) {}
    
    public function show(int $id): array {
        $user = $this->users->findById($id);
        return $user?->toArray() ?? ['error' => 'Not found'];
    }
}
?>
```


## Resources

- [Data Mapper Pattern](https://www.martinfowler.com/eaaCatalog/dataMapper.html) — Martin Fowler's Data Mapper pattern description

---

> 📘 *This lesson is part of the [PHP & Relational Databases](https://stanza.dev/courses/php-databases) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*