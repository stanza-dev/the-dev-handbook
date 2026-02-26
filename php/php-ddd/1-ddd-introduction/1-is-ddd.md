---
source_course: "php-ddd"
source_lesson: "php-ddd-what-is-ddd"
---

# What is Domain-Driven Design?

## Introduction

Domain-Driven Design (DDD) is an approach to software development that centers the design around the core business domain. Introduced by Eric Evans in 2003, DDD helps teams build complex software that accurately models real-world business processes.

## Key Concepts

- **Business experts**
- **Developers**
- **Good fit for DDD:**
- **Probably overkill:**
- **Strategic Design**
- **Tactical Design**

## Real World Context

In production PHP applications, what is domain-driven design? helps teams build maintainable software by providing clear patterns for organizing complex business logic.

## Deep Dive

### The Problem DDD Solves
In many software projects, a disconnect exists between:
- **Business experts** who understand the problem domain
- **Developers** who write the code

This leads to:
- Software that doesn't match business needs
- Miscommunication and misunderstandings
- Technical debt from poor modeling
- Difficulty adapting to business changes

### Core Principles of DDD
### 1. Focus on the Core Domain

Not all parts of your system are equally important. DDD encourages identifying and focusing on:

```
┌─────────────────────────────────────────────────┐
│                Core Domain                       │
│  (Your competitive advantage - invest heavily)   │
│  Example: Pricing algorithm for a trading firm   │
├─────────────────────────────────────────────────┤
│            Supporting Subdomains                 │
│  (Necessary but not differentiating)             │
│  Example: Customer management                    │
├─────────────────────────────────────────────────┤
│            Generic Subdomains                    │
│  (Common problems - buy or use existing)         │
│  Example: Authentication, email sending          │
└─────────────────────────────────────────────────┘
```

### 2. Ubiquitous Language

Create a shared vocabulary between developers and domain experts:

```php
<?php
// BAD: Technical jargon that business doesn't understand
class DataProcessor {
    public function executeTransaction(
        array $payload,
        string $entityId
    ): ResultDTO {
        // ...
    }
}

// GOOD: Language that matches the business domain
class OrderFulfillment {
    public function shipOrder(
        Order $order,
        ShippingMethod $method
    ): Shipment {
        // ...
    }
}
```

### 3. Model-Driven Design

The code should be a direct expression of the domain model:

```php
<?php
// The code mirrors the business concept
final class Order {
    private OrderId $id;
    private CustomerId $customerId;
    private OrderStatus $status;
    private Money $total;
    /** @var OrderLine[] */
    private array $lines;
    
    public function addItem(Product $product, Quantity $quantity): void {
        $this->ensureOrderIsModifiable();
        $this->lines[] = new OrderLine($product, $quantity);
        $this->recalculateTotal();
    }
    
    public function submit(): void {
        $this->ensureHasItems();
        $this->status = OrderStatus::Submitted;
        $this->recordThat(new OrderWasSubmitted($this->id));
    }
    
    public function cancel(CancellationReason $reason): void {
        if (!$this->status->canBeCancelled()) {
            throw new OrderCannotBeCancelled($this->id, $this->status);
        }
        $this->status = OrderStatus::Cancelled;
        $this->recordThat(new OrderWasCancelled($this->id, $reason));
    }
}
```

### Strategic vs Tactical Design
DDD operates at two levels:

**Strategic Design** (high-level):
- How to divide a large system into smaller parts
- How teams collaborate
- Where to invest effort

**Tactical Design** (implementation-level):
- Building blocks like Entities, Value Objects, Aggregates
- How to implement the domain model in code
- Patterns for managing complexity

### When to Use DDD
✅ **Good fit for DDD:**
- Complex business logic
- Long-lived projects with evolving requirements
- Multiple team collaboration
- Business is the competitive advantage

❌ **Probably overkill:**
- Simple CRUD applications
- Short-lived projects
- Well-understood, stable domains
- Technical (non-business) systems

### The DDD Building Blocks
```
Strategic Patterns          Tactical Patterns
───────────────────         ─────────────────
• Bounded Context           • Entity
• Ubiquitous Language       • Value Object
• Context Mapping           • Aggregate
• Subdomain                 • Domain Event
                            • Repository
                            • Domain Service
                            • Factory
```

### A Simple Example
Consider an e-commerce system. A business expert might say:

> "When a customer places an order, we need to check inventory, reserve the items, calculate the total including any applicable discounts, and then process the payment."

DDD approach:

```php
<?php
final class PlaceOrderService {
    public function __construct(
        private InventoryService $inventory,
        private DiscountCalculator $discounts,
        private PaymentGateway $payments,
        private OrderRepository $orders
    ) {}
    
    public function placeOrder(PlaceOrderCommand $command): OrderId {
        // Check and reserve inventory
        $reservations = $this->inventory->reserve(
            $command->items,
            $command->customerId
        );
        
        // Create order with business logic
        $order = Order::place(
            customerId: $command->customerId,
            items: $command->items,
            shippingAddress: $command->shippingAddress
        );
        
        // Apply discounts (domain logic)
        $discount = $this->discounts->calculateFor($order);
        $order->applyDiscount($discount);
        
        // Process payment
        $payment = $this->payments->charge(
            $order->total(),
            $command->paymentMethod
        );
        $order->confirmPayment($payment);
        
        // Persist
        $this->orders->save($order);
        
        return $order->id();
    }
}
```

Notice how the code reads almost like the business description.

## Common Pitfalls

1. **Overcomplicating simple cases** - Not every part of the application needs what is domain-driven design?. Apply it where complexity warrants the investment.
2. **Ignoring the ubiquitous language** - Naming classes and methods without input from domain experts leads to a model that does not reflect the business.
3. **Mixing infrastructure concerns** - Allowing framework dependencies to leak into the domain layer undermines the architectural benefits.

## Best Practices

1. **Start from the domain** - Model the business concepts first, then figure out persistence and infrastructure.
2. **Keep it simple** - Use the simplest pattern that solves the problem. Introduce complexity only when needed.
3. **Collaborate with domain experts** - The model should be shaped by business knowledge, not just technical preferences.

## Summary

- What is Domain-Driven Design? is a fundamental concept in Domain-Driven Design
- Proper implementation leads to more maintainable and expressive code
- Always align your implementation with the ubiquitous language
- Apply these patterns where business complexity justifies the investment

## Resources

- [Domain-Driven Design Reference](https://www.domainlanguage.com/ddd/reference/) — Eric Evans' DDD reference definitions

---

> 📘 *This lesson is part of the [Domain-Driven Design with PHP](https://stanza.dev/courses/php-ddd) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*