---
source_course: "php-ddd"
source_lesson: "php-ddd-entities"
---

# Entities

## Introduction

An **Entity** is a domain object defined by its identity rather than its attributes. Two entities with the same attributes but different identities are considered different objects.

## Key Concepts

- **Continuity**
- **Entity**
- **Equality**
- **Identity**
- **Lifecycle**
- **Mutability**

## Real World Context

In production PHP applications, entities helps teams build maintainable software by providing clear patterns for organizing complex business logic.

## Deep Dive

### Identity vs Equality
Consider a `Customer`:

```php
<?php
// Two customers with same name but different IDs
$customer1 = new Customer(
    id: new CustomerId('cust-001'),
    name: 'John Smith'
);

$customer2 = new Customer(
    id: new CustomerId('cust-002'),
    name: 'John Smith'
);

// These are DIFFERENT entities (different identity)
$customer1->equals($customer2);  // false

// Even if we change attributes, identity remains
$customer1->changeName('Jonathan Smith');
$customer1->id();  // Still 'cust-001'
```

### Implementing Entities
```php
<?php
declare(strict_types=1);

namespace Domain\Order;

final class Order {
    private OrderId $id;
    private CustomerId $customerId;
    private OrderStatus $status;
    private Money $total;
    /** @var OrderLine[] */
    private array $lines = [];
    private \DateTimeImmutable $createdAt;
    private ?\DateTimeImmutable $submittedAt = null;
    
    private function __construct(
        OrderId $id,
        CustomerId $customerId
    ) {
        $this->id = $id;
        $this->customerId = $customerId;
        $this->status = OrderStatus::Draft;
        $this->total = Money::zero();
        $this->createdAt = new \DateTimeImmutable();
    }
    
    /**
     * Named constructor - expresses intent
     */
    public static function create(
        OrderId $id,
        CustomerId $customerId
    ): self {
        return new self($id, $customerId);
    }
    
    /**
     * Reconstitute from persistence - no validation
     */
    public static function reconstitute(
        OrderId $id,
        CustomerId $customerId,
        OrderStatus $status,
        array $lines,
        \DateTimeImmutable $createdAt,
        ?\DateTimeImmutable $submittedAt
    ): self {
        $order = new self($id, $customerId);
        $order->status = $status;
        $order->lines = $lines;
        $order->createdAt = $createdAt;
        $order->submittedAt = $submittedAt;
        $order->recalculateTotal();
        return $order;
    }
    
    public function id(): OrderId {
        return $this->id;
    }
    
    public function addLine(
        ProductId $productId,
        Quantity $quantity,
        Money $unitPrice
    ): void {
        $this->assertDraft();
        
        $this->lines[] = new OrderLine(
            productId: $productId,
            quantity: $quantity,
            unitPrice: $unitPrice
        );
        
        $this->recalculateTotal();
    }
    
    public function submit(): void {
        $this->assertDraft();
        $this->assertHasLines();
        
        $this->status = OrderStatus::Submitted;
        $this->submittedAt = new \DateTimeImmutable();
    }
    
    public function cancel(): void {
        if (!$this->status->canBeCancelled()) {
            throw CannotCancelOrder::becauseStatusIs($this->status);
        }
        
        $this->status = OrderStatus::Cancelled;
    }
    
    public function equals(Order $other): bool {
        return $this->id->equals($other->id);
    }
    
    private function assertDraft(): void {
        if (!$this->status->isDraft()) {
            throw InvalidOrderOperation::cannotModifyNonDraft();
        }
    }
    
    private function assertHasLines(): void {
        if (empty($this->lines)) {
            throw InvalidOrderOperation::cannotSubmitEmpty();
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

### Entity Characteristics
| Characteristic | Description |
|----------------|-------------|
| **Identity** | Unique identifier that never changes |
| **Continuity** | Same entity over time despite attribute changes |
| **Mutability** | Attributes can change through behavior methods |
| **Lifecycle** | Created, modified, possibly deleted |
| **Equality** | Based on identity, not attributes |

### Entity Identity
```php
<?php
// Value Object for identity
final readonly class OrderId {
    private function __construct(
        private string $value
    ) {
        if (empty($value)) {
            throw new InvalidArgumentException('OrderId cannot be empty');
        }
    }
    
    public static function generate(): self {
        return new self(Uuid::uuid4()->toString());
    }
    
    public static function fromString(string $id): self {
        return new self($id);
    }
    
    public function toString(): string {
        return $this->value;
    }
    
    public function equals(OrderId $other): bool {
        return $this->value === $other->value;
    }
}
```

### Entity Design Guidelines
### 1. Encapsulate State Changes

```php
<?php
// BAD: Exposing internal state
class Order {
    public OrderStatus $status;  // Can be changed externally!
    public array $lines;          // Can be modified directly!
}

// GOOD: Controlled changes through behavior
final class Order {
    private OrderStatus $status;
    private array $lines;
    
    public function submit(): void {  // Behavior method
        $this->status = OrderStatus::Submitted;
    }
    
    public function addLine(OrderLine $line): void {  // Controlled modification
        $this->lines[] = $line;
    }
}
```

### 2. Enforce Invariants

```php
<?php
final class Order {
    // Invariant: An order must always have a valid total
    // Invariant: Order cannot be modified after submission
    
    public function addLine(OrderLine $line): void {
        // Check invariant before modification
        if ($this->status !== OrderStatus::Draft) {
            throw new DomainException(
                'Cannot add lines to a non-draft order'
            );
        }
        
        $this->lines[] = $line;
        $this->recalculateTotal();  // Maintain invariant
    }
}
```

### 3. Use Named Constructors

```php
<?php
final class Customer {
    private function __construct(
        private CustomerId $id,
        private string $name,
        private CustomerType $type
    ) {}
    
    // Clear intent: registering a new customer
    public static function register(
        CustomerId $id,
        string $name,
        string $email
    ): self {
        $customer = new self($id, $name, CustomerType::Regular);
        $customer->recordEvent(new CustomerRegistered($id, $name, $email));
        return $customer;
    }
    
    // Clear intent: importing from legacy system
    public static function importFromLegacy(
        CustomerId $id,
        array $legacyData
    ): self {
        return new self(
            $id,
            $legacyData['name'],
            CustomerType::fromLegacyCode($legacyData['type_code'])
        );
    }
}
```

## Common Pitfalls

1. **Overcomplicating simple cases** - Not every part of the application needs entities. Apply it where complexity warrants the investment.
2. **Ignoring the ubiquitous language** - Naming classes and methods without input from domain experts leads to a model that does not reflect the business.
3. **Mixing infrastructure concerns** - Allowing framework dependencies to leak into the domain layer undermines the architectural benefits.

## Best Practices

1. **Start from the domain** - Model the business concepts first, then figure out persistence and infrastructure.
2. **Keep it simple** - Use the simplest pattern that solves the problem. Introduce complexity only when needed.
3. **Collaborate with domain experts** - The model should be shaped by business knowledge, not just technical preferences.

## Summary

- Entities is a fundamental concept in Domain-Driven Design
- Proper implementation leads to more maintainable and expressive code
- Always align your implementation with the ubiquitous language
- Apply these patterns where business complexity justifies the investment

## Resources

- [PHP Manual - Classes and Objects](https://www.php.net/manual/en/language.oop5.php) — PHP OOP reference for implementing entities

---

> 📘 *This lesson is part of the [Domain-Driven Design with PHP](https://stanza.dev/courses/php-ddd) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*