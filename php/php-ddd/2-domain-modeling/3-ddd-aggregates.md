---
source_course: "php-ddd"
source_lesson: "php-ddd-aggregates"
---

# Aggregates

## Introduction

An **Aggregate** is a cluster of domain objects treated as a single unit for data changes. Each aggregate has a root entity (the **Aggregate Root**) that controls all access to the aggregate.

## Key Concepts

- **Aggregate**
- **Aggregate Root**

## Real World Context

In production PHP applications, aggregates helps teams build maintainable software by providing clear patterns for organizing complex business logic.

## Deep Dive

### Why Aggregates?
Without aggregates, maintaining consistency is hard:

```php
<?php
// PROBLEM: Inconsistent state
$order = $orderRepo->find($orderId);
$orderLine = $orderLineRepo->find($lineId);

// Who ensures the order total stays correct?
$orderLine->changeQuantity(5);
$orderLineRepo->save($orderLine);
// Order total is now wrong!
```

With aggregates:

```php
<?php
// SOLUTION: Aggregate controls consistency
$order = $orderRepo->find($orderId);

// All changes go through the aggregate root
$order->updateLineQuantity($lineId, 5);
// Aggregate ensures total is recalculated

$orderRepo->save($order);  // Save entire aggregate
```

### Aggregate Design
```
┌─────────────────────────────────────────────┐
│              ORDER AGGREGATE                │
│  ┌───────────────────────────────────────┐  │
│  │         Order (Aggregate Root)        │  │
│  │  - id: OrderId                        │  │
│  │  - status: OrderStatus                │  │
│  │  - total: Money                       │  │
│  └───────────────────────────────────────┘  │
│       │                                     │
│       │ contains                            │
│       ▼                                     │
│  ┌─────────────────┐  ┌─────────────────┐   │
│  │   OrderLine     │  │   OrderLine     │   │
│  │  - productId    │  │  - productId    │   │
│  │  - quantity     │  │  - quantity     │   │
│  │  - unitPrice    │  │  - unitPrice    │   │
│  └─────────────────┘  └─────────────────┘   │
│                                             │
│  Boundary: Only Order can be accessed       │
│            from outside the aggregate       │
└─────────────────────────────────────────────┘
```

### Implementing an Aggregate
```php
<?php
declare(strict_types=1);

namespace Domain\Ordering;

/**
 * Order Aggregate Root
 */
final class Order {
    private OrderId $id;
    private CustomerId $customerId;
    private OrderStatus $status;
    private Money $total;
    /** @var array<string, OrderLine> */
    private array $lines = [];
    private array $domainEvents = [];
    
    private function __construct(
        OrderId $id,
        CustomerId $customerId
    ) {
        $this->id = $id;
        $this->customerId = $customerId;
        $this->status = OrderStatus::Draft;
        $this->total = Money::zero();
    }
    
    public static function place(
        OrderId $id,
        CustomerId $customerId,
        array $items
    ): self {
        $order = new self($id, $customerId);
        
        foreach ($items as $item) {
            $order->addLine(
                $item['productId'],
                $item['quantity'],
                $item['unitPrice']
            );
        }
        
        $order->recordEvent(new OrderWasPlaced($id, $customerId));
        
        return $order;
    }
    
    /**
     * Add a line item to the order
     */
    public function addLine(
        ProductId $productId,
        Quantity $quantity,
        Money $unitPrice
    ): void {
        $this->assertModifiable();
        
        $line = new OrderLine($productId, $quantity, $unitPrice);
        $this->lines[$productId->toString()] = $line;
        
        $this->recalculateTotal();
    }
    
    /**
     * Update quantity of an existing line
     */
    public function updateLineQuantity(
        ProductId $productId,
        Quantity $newQuantity
    ): void {
        $this->assertModifiable();
        
        $key = $productId->toString();
        if (!isset($this->lines[$key])) {
            throw new LineNotFound($productId);
        }
        
        if ($newQuantity->isZero()) {
            $this->removeLine($productId);
            return;
        }
        
        $this->lines[$key] = $this->lines[$key]->withQuantity($newQuantity);
        $this->recalculateTotal();
    }
    
    /**
     * Remove a line item
     */
    public function removeLine(ProductId $productId): void {
        $this->assertModifiable();
        
        $key = $productId->toString();
        if (!isset($this->lines[$key])) {
            throw new LineNotFound($productId);
        }
        
        unset($this->lines[$key]);
        $this->recalculateTotal();
    }
    
    /**
     * Submit the order for processing
     */
    public function submit(): void {
        $this->assertModifiable();
        
        if (empty($this->lines)) {
            throw CannotSubmitOrder::becauseItIsEmpty();
        }
        
        $this->status = OrderStatus::Submitted;
        $this->recordEvent(new OrderWasSubmitted($this->id, $this->total));
    }
    
    /**
     * Cancel the order
     */
    public function cancel(CancellationReason $reason): void {
        if (!$this->status->canBeCancelled()) {
            throw CannotCancelOrder::becauseStatusIs($this->status);
        }
        
        $this->status = OrderStatus::Cancelled;
        $this->recordEvent(new OrderWasCancelled($this->id, $reason));
    }
    
    // Getters
    public function id(): OrderId {
        return $this->id;
    }
    
    public function status(): OrderStatus {
        return $this->status;
    }
    
    public function total(): Money {
        return $this->total;
    }
    
    /** @return OrderLine[] */
    public function lines(): array {
        return array_values($this->lines);
    }
    
    // Event handling
    public function pullDomainEvents(): array {
        $events = $this->domainEvents;
        $this->domainEvents = [];
        return $events;
    }
    
    private function recordEvent(object $event): void {
        $this->domainEvents[] = $event;
    }
    
    // Invariant enforcement
    private function assertModifiable(): void {
        if ($this->status !== OrderStatus::Draft) {
            throw OrderNotModifiable::becauseStatusIs($this->status);
        }
    }
    
    private function recalculateTotal(): void {
        $this->total = array_reduce(
            $this->lines,
            fn(Money $sum, OrderLine $line) => $sum->add($line->subtotal()),
            Money::zero()
        );
    }
}
```

### Aggregate Rules
### 1. Reference by Identity

```php
<?php
// BAD: Direct reference to another aggregate
class Order {
    private Customer $customer;  // Full Customer object
}

// GOOD: Reference by identity
class Order {
    private CustomerId $customerId;  // Just the ID
}
```

### 2. One Transaction = One Aggregate

```php
<?php
// BAD: Modifying multiple aggregates in one transaction
public function transferMoney(Account $from, Account $to, Money $amount): void {
    $from->withdraw($amount);
    $to->deposit($amount);
    // What if saving $to fails? $from already withdrawn!
}

// GOOD: Use domain events for eventual consistency
public function withdraw(Account $account, Money $amount): void {
    $account->withdraw($amount);
    $this->accountRepo->save($account);
    // Dispatch event: MoneyWithdrawn
    // Another handler will deposit to target account
}
```

### 3. Keep Aggregates Small

```php
<?php
// BAD: Huge aggregate with everything
class Order {
    private Customer $customer;
    private array $lines;
    private array $payments;
    private array $shipments;
    private array $reviews;
    // Too much!
}

// GOOD: Focused aggregate
class Order {
    private CustomerId $customerId;  // Reference only
    private array $lines;
    // Payments, Shipments are separate aggregates
}
```

### The Order Line (Internal Entity)
```php
<?php
/**
 * OrderLine - Entity within Order Aggregate
 * Cannot exist outside the Order aggregate
 */
final readonly class OrderLine {
    public function __construct(
        private ProductId $productId,
        private Quantity $quantity,
        private Money $unitPrice
    ) {}
    
    public function productId(): ProductId {
        return $this->productId;
    }
    
    public function quantity(): Quantity {
        return $this->quantity;
    }
    
    public function subtotal(): Money {
        return $this->unitPrice->multiply($this->quantity->value());
    }
    
    public function withQuantity(Quantity $quantity): self {
        return new self(
            $this->productId,
            $quantity,
            $this->unitPrice
        );
    }
}
```

## Common Pitfalls

1. **Overcomplicating simple cases** - Not every part of the application needs aggregates. Apply it where complexity warrants the investment.
2. **Ignoring the ubiquitous language** - Naming classes and methods without input from domain experts leads to a model that does not reflect the business.
3. **Mixing infrastructure concerns** - Allowing framework dependencies to leak into the domain layer undermines the architectural benefits.

## Best Practices

1. **Start from the domain** - Model the business concepts first, then figure out persistence and infrastructure.
2. **Keep it simple** - Use the simplest pattern that solves the problem. Introduce complexity only when needed.
3. **Collaborate with domain experts** - The model should be shaped by business knowledge, not just technical preferences.

## Summary

- Aggregates is a fundamental concept in Domain-Driven Design
- Proper implementation leads to more maintainable and expressive code
- Always align your implementation with the ubiquitous language
- Apply these patterns where business complexity justifies the investment

## Resources

- [Effective Aggregate Design](https://www.dddcommunity.org/library/vernon_2011/) — Vaughn Vernon's guide to aggregate design

---

> 📘 *This lesson is part of the [Domain-Driven Design with PHP](https://stanza.dev/courses/php-ddd) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*