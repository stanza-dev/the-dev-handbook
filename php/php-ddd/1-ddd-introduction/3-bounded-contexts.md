---
source_course: "php-ddd"
source_lesson: "php-ddd-bounded-contexts"
---

# Bounded Contexts

A **Bounded Context** is a central pattern in DDD that defines clear boundaries within which a particular domain model applies. It's where the ubiquitous language has a specific, unambiguous meaning.

## Why Bounded Contexts?

Large systems can't have a single, unified model:

```
"Customer" in different contexts:

┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
│     SALES       │  │    SHIPPING     │  │    BILLING      │
├─────────────────┤  ├─────────────────┤  ├─────────────────┤
│ Customer:       │  │ Customer:       │  │ Customer:       │
│ - Name          │  │ - Address       │  │ - Payment Info  │
│ - Credit Limit  │  │ - Phone         │  │ - Tax ID        │
│ - Sales Rep     │  │ - Delivery Pref │  │ - Billing Addr  │
│ - Discount Tier │  │                 │  │ - Credit Terms  │
└─────────────────┘  └─────────────────┘  └─────────────────┘
```

Each context has its own model of Customer with different attributes and behaviors.

## Identifying Bounded Contexts

Look for:

1. **Different meanings for the same term**
2. **Different teams or departments**
3. **Different business processes**
4. **Different data requirements**

```php
<?php
// Sales Context - cares about purchasing behavior
namespace Sales;

final class Customer {
    private CustomerId $id;
    private string $name;
    private Money $creditLimit;
    private DiscountTier $discountTier;
    private ?SalesRepId $assignedRep;
    
    public function canPurchase(Money $amount): bool {
        return $this->creditLimit->isGreaterThanOrEqual($amount);
    }
    
    public function getDiscountPercentage(): Percentage {
        return $this->discountTier->asPercentage();
    }
}

// Shipping Context - cares about delivery
namespace Shipping;

final class Recipient {  // Note: different name, same real-world person
    private RecipientId $id;
    private Address $shippingAddress;
    private PhoneNumber $contactPhone;
    private DeliveryInstructions $instructions;
    
    public function requiresSignature(): bool {
        return $this->instructions->requiresSignature();
    }
}
```

## Context Mapping

Bounded Contexts must interact. Context Maps define these relationships:

### Partnership

Two teams succeed or fail together:

```php
<?php
// Both contexts evolve together
namespace Sales;
use Shipping\ShipmentScheduler;

class OrderService {
    public function __construct(
        private ShipmentScheduler $shipping  // Direct dependency OK
    ) {}
}
```

### Customer-Supplier

Upstream (supplier) context provides what downstream (customer) needs:

```php
<?php
// Supplier: Inventory context provides stock info
namespace Inventory;

interface StockQueryService {
    public function getAvailableStock(ProductId $productId): Quantity;
    public function reserveStock(ProductId $productId, Quantity $qty): Reservation;
}

// Customer: Sales context uses inventory
namespace Sales;

class OrderService {
    public function __construct(
        private \Inventory\StockQueryService $inventory
    ) {}
}
```

### Anti-Corruption Layer (ACL)

Protect your domain from external/legacy models:

```php
<?php
namespace Sales\Infrastructure\Acl;

use Sales\Domain\Customer;
use Sales\Domain\CustomerId;
use LegacyCRM\ClientRecord;  // External system

class LegacyCrmCustomerAdapter implements CustomerRepository {
    public function __construct(
        private LegacyCRM\Database $legacyDb
    ) {}
    
    public function findById(CustomerId $id): ?Customer {
        // Fetch from legacy system
        $record = $this->legacyDb->fetchClient($id->toString());
        
        if ($record === null) {
            return null;
        }
        
        // Translate to our domain model
        return $this->translateToDomain($record);
    }
    
    private function translateToDomain(ClientRecord $record): Customer {
        return new Customer(
            id: new CustomerId($record->CLIENT_ID),
            name: $record->CLIENT_NAME ?? 'Unknown',
            creditLimit: Money::fromCents(
                (int)($record->CREDIT_LIM * 100)
            ),
            // Map legacy codes to domain concepts
            discountTier: $this->mapDiscountCode($record->DISC_CODE)
        );
    }
    
    private function mapDiscountCode(?string $code): DiscountTier {
        return match($code) {
            'A' => DiscountTier::Premium,
            'B' => DiscountTier::Standard,
            default => DiscountTier::None,
        };
    }
}
```

### Published Language

Use a shared, documented format for integration:

```php
<?php
// Published contract (e.g., OpenAPI, JSON Schema)
namespace Contracts;

final class OrderPlacedEvent {
    public const SCHEMA = [
        'type' => 'object',
        'properties' => [
            'orderId' => ['type' => 'string', 'format' => 'uuid'],
            'customerId' => ['type' => 'string', 'format' => 'uuid'],
            'totalAmount' => [
                'type' => 'object',
                'properties' => [
                    'amount' => ['type' => 'integer'],
                    'currency' => ['type' => 'string']
                ]
            ],
            'occurredAt' => ['type' => 'string', 'format' => 'date-time']
        ],
        'required' => ['orderId', 'customerId', 'totalAmount', 'occurredAt']
    ];
}
```

## PHP Project Structure

```
src/
├── Sales/                      # Bounded Context
│   ├── Domain/
│   │   ├── Customer.php
│   │   ├── Order.php
│   │   └── OrderRepository.php
│   ├── Application/
│   │   └── PlaceOrderService.php
│   └── Infrastructure/
│       ├── Persistence/
│       └── Acl/
│
├── Shipping/                   # Bounded Context
│   ├── Domain/
│   │   ├── Recipient.php
│   │   └── Shipment.php
│   └── ...
│
├── Billing/                    # Bounded Context
│   └── ...
│
└── SharedKernel/               # Shared code across contexts
    ├── Domain/
    │   ├── Money.php
    │   └── EventInterface.php
    └── Infrastructure/
        └── EventBus.php
```

## Shared Kernel

Code shared between contexts—use sparingly:

```php
<?php
namespace SharedKernel\Domain;

// Shared across all contexts
final readonly class Money {
    public function __construct(
        private int $cents,
        private Currency $currency
    ) {}
    
    public static function USD(int $cents): self {
        return new self($cents, Currency::USD);
    }
    
    public function add(Money $other): self {
        $this->ensureSameCurrency($other);
        return new self(
            $this->cents + $other->cents,
            $this->currency
        );
    }
    
    // ...
}
```

⚠️ Shared Kernel creates coupling. Keep it minimal and stable.

## Resources

- [Bounded Context Explained](https://martinfowler.com/bliki/BoundedContext.html) — Martin Fowler's explanation of Bounded Contexts

---

> 📘 *This lesson is part of the [Domain-Driven Design with PHP](https://stanza.dev/courses/php-ddd) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*