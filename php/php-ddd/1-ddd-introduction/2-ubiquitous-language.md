---
source_course: "php-ddd"
source_lesson: "php-ddd-ubiquitous-language"
---

# Ubiquitous Language

The **Ubiquitous Language** is a shared vocabulary used by everyone on the team—developers, domain experts, product owners, and stakeholders. It forms the foundation of effective DDD.

## Why Language Matters

Miscommunication is expensive:

```
Business Expert: "The customer account is suspended."
Developer thinks: Customer can't log in
Business actually means: Customer can log in but can't make purchases

→ Wrong feature built → Wasted time → Frustrated users
```

## Building the Language

### 1. Discovery Through Conversation

```
Developer: "So when a user creates an order..."
Expert: "Actually, we call them 'customers', and they 'place' orders,
        they don't 'create' them."
Developer: "Got it! And what happens when they place an order?"
Expert: "The order is 'pending' until we 'confirm' it."
```

### 2. Document the Language

Create a glossary that everyone references:

```markdown
## Order Management Glossary

**Customer** - A person or organization that purchases products.
Not "user" or "client".

**Order** - A request by a customer to purchase one or more products.
- Can be: Pending, Confirmed, Shipped, Delivered, Cancelled

**Place an Order** - The act of a customer submitting an order.
Not "create" or "submit".

**Confirm an Order** - When inventory is reserved and payment
is authorized. Not "approve" or "process".

**Line Item** - A single product entry in an order with quantity.
Not "order item" or "product line".
```

### 3. Enforce in Code

```php
<?php
// Language violations - confusing and incorrect terms
class User {  // Should be Customer
    public function createOrder() {}  // Should be placeOrder
}

class OrderItem {}  // Should be LineItem

function processOrder() {}  // What does "process" mean?

// Language-compliant code
final class Customer {
    public function placeOrder(Cart $cart): Order {
        return Order::place(
            customerId: $this->id,
            lineItems: $cart->toLineItems()
        );
    }
}

final class LineItem {
    public function __construct(
        public readonly ProductId $productId,
        public readonly Quantity $quantity,
        public readonly Money $unitPrice
    ) {}
}
```

## Language in Different Contexts

The same word can mean different things in different contexts:

```php
<?php
// In the Sales context
namespace Sales\Domain;

final class Customer {
    private CustomerId $id;
    private CustomerName $name;
    private CreditLimit $creditLimit;
    private PaymentTerms $paymentTerms;
    
    public function canPlaceOrder(Money $orderTotal): bool {
        return $this->creditLimit->covers($orderTotal);
    }
}

// In the Shipping context - same "Customer" but different data
namespace Shipping\Domain;

final class Customer {
    private CustomerId $id;
    private ShippingAddress $address;
    private DeliveryPreferences $preferences;
    
    public function preferredDeliverySlot(): TimeSlot {
        return $this->preferences->preferredSlot();
    }
}
```

## Patterns for Maintaining Language

### Use Static Analysis

```php
<?php
// Custom PHPStan rule to enforce language
namespace App\PHPStan;

use PHPStan\Rules\Rule;

class ForbiddenTermsRule implements Rule {
    private const FORBIDDEN_TERMS = [
        'User' => 'Use "Customer" instead',
        'createOrder' => 'Use "placeOrder" instead',
        'OrderItem' => 'Use "LineItem" instead',
    ];
    
    // Implementation...
}
```

### Code Reviews

Include language checks in your review process:

- Does this PR use terms from our glossary?
- Are there any new domain concepts that should be added?
- Would a domain expert understand this code?

### Living Documentation

```php
<?php
/**
 * Represents a Customer's request to purchase products.
 * 
 * Business Rules:
 * - An Order must have at least one LineItem
 * - An Order can only be cancelled before it's shipped
 * - Discounts are applied at the Order level, not LineItem
 * 
 * @see docs/glossary.md#order
 */
final class Order {
    // ...
}
```

## Common Language Anti-Patterns

### 1. Technical Leakage

```php
<?php
// BAD: Technical terms in domain code
class OrderEntity {  // "Entity" is technical
    private int $id;  // Should be OrderId value object
    
    public function persist(): void {}  // Domain doesn't persist!
}

// GOOD: Pure domain language
final class Order {
    private OrderId $id;
    
    public function confirm(): void {
        // Domain behavior only
    }
}
```

### 2. Generic Names

```php
<?php
// BAD: Generic, meaningless names
class Manager {}
class Handler {}
class Processor {}
class Helper {}

// GOOD: Specific domain terms
class OrderFulfillmentService {}
class PricingCalculator {}
class InventoryAllocator {}
```

### 3. Abbreviated Terms

```php
<?php
// BAD: Abbreviations hurt understanding
$custOrdLn = new CustOrdLn();
$po = $repo->getPO($id);

// GOOD: Full, clear terms
$lineItem = new LineItem();
$purchaseOrder = $repository->findPurchaseOrder($id);
```

## Evolving the Language

The ubiquitous language isn't static—it evolves:

1. **New concepts emerge** as business grows
2. **Terms get refined** with better understanding
3. **Old terms retire** when business changes

```php
<?php
// Version 1: Simple status
enum OrderStatus: string {
    case Pending = 'pending';
    case Complete = 'complete';
}

// Version 2: Business learned more nuance is needed
enum OrderStatus: string {
    case Draft = 'draft';  // New: orders can be saved without submitting
    case Placed = 'placed';  // Renamed from Pending
    case Confirmed = 'confirmed';  // New: separate confirmation step
    case Shipped = 'shipped';  // New: tracking shipping
    case Delivered = 'delivered';  // Renamed from Complete
    case Cancelled = 'cancelled';
}
```

## Resources

- [Developing the Ubiquitous Language](https://martinfowler.com/bliki/UbiquitousLanguage.html) — Martin Fowler on Ubiquitous Language

---

> 📘 *This lesson is part of the [Domain-Driven Design with PHP](https://stanza.dev/courses/php-ddd) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*