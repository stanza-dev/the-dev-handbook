---
source_course: "php-ddd"
source_lesson: "php-ddd-domain-services"
---

# Domain Services

A **Domain Service** encapsulates domain logic that doesn't naturally fit within a single Entity or Value Object. It represents an operation or process from the ubiquitous language.

## When to Use Domain Services

Use a Domain Service when:

1. **Operation involves multiple aggregates**
2. **Stateless calculation or transformation**
3. **The behavior doesn't belong to any entity**

```php
<?php
// BAD: Forcing behavior into an entity
class Order {
    public function calculateShipping(
        Address $from,
        ShippingCarrier $carrier
    ): Money {
        // Order shouldn't know shipping calculation logic!
    }
}

// GOOD: Domain Service for cross-cutting concerns
final class ShippingCostCalculator {
    public function calculate(
        Order $order,
        Address $destination,
        ShippingMethod $method
    ): Money {
        $weight = $order->totalWeight();
        $distance = $this->distanceCalculator->between(
            $this->warehouse->address(),
            $destination
        );
        
        return $method->calculateCost($weight, $distance);
    }
}
```

## Domain Service Examples

### Pricing Calculator

```php
<?php
namespace Domain\Pricing;

final class PricingService {
    public function __construct(
        private DiscountPolicyRepository $policies,
        private TaxCalculator $taxCalculator
    ) {}
    
    public function calculatePrice(
        Product $product,
        Customer $customer,
        Quantity $quantity
    ): PricingResult {
        $basePrice = $product->price()->multiply($quantity->value());
        
        // Apply customer-specific discounts
        $discount = $this->calculateDiscount(
            $customer,
            $product,
            $basePrice
        );
        
        $discountedPrice = $basePrice->subtract($discount);
        
        // Calculate tax
        $tax = $this->taxCalculator->calculate(
            $discountedPrice,
            $product->taxCategory(),
            $customer->taxJurisdiction()
        );
        
        return new PricingResult(
            basePrice: $basePrice,
            discount: $discount,
            tax: $tax,
            total: $discountedPrice->add($tax)
        );
    }
    
    private function calculateDiscount(
        Customer $customer,
        Product $product,
        Money $basePrice
    ): Money {
        $policies = $this->policies->findApplicable(
            $customer->tier(),
            $product->category()
        );
        
        $totalDiscount = Money::zero();
        
        foreach ($policies as $policy) {
            $discount = $policy->calculate($basePrice);
            $totalDiscount = $totalDiscount->add($discount);
        }
        
        return $totalDiscount;
    }
}
```

### Transfer Service

```php
<?php
namespace Domain\Banking;

final class MoneyTransferService {
    public function transfer(
        Account $source,
        Account $destination,
        Money $amount
    ): TransferResult {
        // Validate business rules
        if (!$source->canWithdraw($amount)) {
            return TransferResult::failed(
                TransferFailure::InsufficientFunds
            );
        }
        
        if (!$destination->canReceive($amount)) {
            return TransferResult::failed(
                TransferFailure::DestinationRestricted
            );
        }
        
        // Create transfer record
        $transfer = Transfer::initiate(
            TransferId::generate(),
            $source->id(),
            $destination->id(),
            $amount
        );
        
        // Note: Actual account modifications happen via events
        // to maintain aggregate boundaries
        
        return TransferResult::initiated($transfer);
    }
}
```

## Domain Service Characteristics

| Characteristic | Description |
|----------------|-------------|
| **Stateless** | No instance state, operates on parameters |
| **Domain language** | Named using ubiquitous language |
| **Interface-based** | Often defined as interface, implemented in infrastructure |
| **Pure domain logic** | No infrastructure concerns |

## Domain vs Application Services

```php
<?php
// DOMAIN SERVICE: Pure business logic
namespace Domain\Ordering;

final class OrderPricingService {
    public function calculateTotal(Order $order): Money {
        // Pure calculation, no I/O
    }
}

// APPLICATION SERVICE: Orchestrates use case
namespace Application\Ordering;

final class PlaceOrderHandler {
    public function __construct(
        private OrderRepository $orders,
        private OrderPricingService $pricing,
        private EventBus $events
    ) {}
    
    public function handle(PlaceOrderCommand $command): OrderId {
        // Coordinates: loading, domain logic, saving, events
        $order = Order::place(...);
        $total = $this->pricing->calculateTotal($order);
        $this->orders->save($order);
        $this->events->dispatch($order->pullDomainEvents());
        return $order->id();
    }
}
```

## Resources

- [Domain Services vs Application Services](https://enterprisecraftsmanship.com/posts/domain-vs-application-services/) — Understanding the difference between service types

---

> 📘 *This lesson is part of the [Domain-Driven Design with PHP](https://stanza.dev/courses/php-ddd) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*