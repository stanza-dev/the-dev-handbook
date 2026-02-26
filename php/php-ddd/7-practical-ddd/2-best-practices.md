---
source_course: "php-ddd"
source_lesson: "php-ddd-ddd-best-practices"
---

# DDD Best Practices & Common Pitfalls

## Introduction

After learning the patterns, let's discuss practical wisdom for successful DDD adoption.

## Key Concepts

- **Anemic Domain Model** - an anti-pattern where entities lack behavior
- **Bounded Context** - an explicit boundary within which a domain model applies
- **Ubiquitous Language** - the shared vocabulary between developers and domain experts
- **Aggregate Size** - keeping aggregates small to minimize transaction conflicts

## Real World Context

In production PHP applications, ddd best practices & pitfalls helps teams build maintainable software by providing clear patterns for organizing complex business logic.

## Deep Dive

## Best Practices
### 1. Start with the Domain, Not the Database

```php
<?php
// DON'T: Design database first, then model
CREATE TABLE orders (...);  // Database first
class Order extends Model {}  // ORM entity second

// DO: Design domain model first
final class Order {  // Rich domain model
    public function submit(): void { ... }
}
// Then figure out persistence
```

### 2. Make Implicit Concepts Explicit

```php
<?php
// IMPLICIT: Hidden in primitive types
class Order {
    private string $status;  // What values are valid?
    private float $discount;  // Percentage? Amount? Currency?
}

// EXPLICIT: Concepts named and enforced
class Order {
    private OrderStatus $status;  // Enum with valid states
    private DiscountPolicy $discount;  // Typed, with rules
}

enum OrderStatus: string {
    case Draft = 'draft';
    case Submitted = 'submitted';
    case Confirmed = 'confirmed';
    
    public function canBeCancelled(): bool {
        return $this !== self::Confirmed;
    }
}
```

### 3. Keep Aggregates Small

```php
<?php
// TOO BIG: Everything in one aggregate
class Order {
    private Customer $customer;      // Should be ID only
    private array $payments;         // Separate aggregate
    private array $shipments;        // Separate aggregate
    private array $returns;          // Separate aggregate
    private array $reviews;          // Different context
}

// RIGHT SIZE: Focused on core invariants
class Order {
    private CustomerId $customerId;  // Reference only
    private array $lines;            // Part of order invariants
    private Money $total;            // Maintained by aggregate
}
```

### 4. Use Domain Events for Cross-Aggregate Communication

```php
<?php
// BAD: Direct coupling between aggregates
class OrderService {
    public function submit(Order $order): void {
        $order->submit();
        
        // Direct manipulation of other aggregates
        $this->inventory->reduce(...);
        $this->customer->addPoints(...);
        $this->analytics->track(...);
    }
}

// GOOD: Events decouple aggregates
class OrderService {
    public function submit(Order $order): void {
        $order->submit();
        $this->orders->save($order);
        
        // Events handled by separate listeners
        $this->events->dispatch($order->pullDomainEvents());
    }
}

// Listeners handle side effects
class ReduceInventoryOnOrderSubmitted {
    public function __invoke(OrderSubmitted $event): void {
        // Handle in separate transaction
    }
}
```

### 5. Protect Invariants in Aggregates

```php
<?php
class Order {
    // Invariant: Total must always be correct
    // Invariant: Cannot modify after submission
    
    public function addLine(OrderLine $line): void {
        $this->assertModifiable();  // Check invariant
        $this->lines[] = $line;
        $this->recalculateTotal();  // Maintain invariant
    }
    
    private function recalculateTotal(): void {
        // Always kept in sync
        $this->total = array_reduce(
            $this->lines,
            fn($sum, $line) => $sum->add($line->subtotal()),
            Money::zero()
        );
    }
}
```

## Common Pitfalls
### 1. Anemic Domain Model

```php
<?php
// ANEMIC: No behavior, just data
class Order {
    public string $status;
    public array $lines;
    public float $total;
}

class OrderService {
    public function submit(Order $order): void {
        if (empty($order->lines)) {
            throw new Exception('...');
        }
        $order->status = 'submitted';  // Logic outside entity
    }
}

// RICH: Behavior encapsulated
class Order {
    public function submit(): void {
        $this->assertHasLines();
        $this->status = OrderStatus::Submitted;
    }
}
```

### 2. Over-Engineering

```php
<?php
// OVER-ENGINEERED: DDD for simple CRUD
class UserSettings {
    // Does this really need DDD?
    // Sometimes a simple model is enough
}

// Apply DDD where complexity exists
// Use simpler patterns for simple features
```

### 3. Ignoring Bounded Contexts

```php
<?php
// WRONG: One model for everything
class Product {
    // Catalog concerns
    private string $description;
    private array $images;
    
    // Inventory concerns
    private int $stockLevel;
    private string $warehouseLocation;
    
    // Pricing concerns
    private Money $basePrice;
    private array $discountRules;
}

// RIGHT: Different models per context
namespace Catalog { class Product { /* display info */ } }
namespace Inventory { class StockItem { /* stock info */ } }
namespace Pricing { class PricedProduct { /* pricing */ } }
```

### When NOT to Use DDD
- Simple CRUD applications
- Prototypes and MVPs
- Well-understood, stable domains
- Small teams with simple requirements
- Short-lived projects

### Summary Checklist
✅ Domain experts involved in modeling
✅ Ubiquitous language documented and enforced
✅ Bounded contexts identified
✅ Aggregates keep invariants consistent
✅ Repositories abstract persistence
✅ Domain events for cross-aggregate communication
✅ Application services orchestrate use cases
✅ Infrastructure depends on domain, not vice versa

## Resources

- [Domain-Driven Design Quickly](https://www.infoq.com/minibooks/domain-driven-design-quickly/) — Free DDD summary book from InfoQ

---

> 📘 *This lesson is part of the [Domain-Driven Design with PHP](https://stanza.dev/courses/php-ddd) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*