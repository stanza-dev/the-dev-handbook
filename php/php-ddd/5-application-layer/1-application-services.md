---
source_course: "php-ddd"
source_lesson: "php-ddd-application-services"
---

# Application Services

## Introduction

**Application Services** orchestrate use cases by coordinating domain objects, repositories, and infrastructure services. They're the entry point for application operations.

## Key Concepts

- **Application Services**

## Real World Context

In production PHP applications, application services helps teams build maintainable software by providing clear patterns for organizing complex business logic.

## Deep Dive

### Role of Application Services
```
┌─────────────────────────────────────────────────────┐
│                  Presentation Layer                  │
│              (Controllers, CLI, API)                 │
└─────────────────────────┬───────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────┐
│                Application Layer                     │
│  ┌─────────────────────────────────────────────┐    │
│  │          Application Services               │    │
│  │  • Orchestrate use cases                    │    │
│  │  • Transaction management                   │    │
│  │  • Security/authorization                   │    │
│  │  • Convert DTOs to domain objects           │    │
│  └─────────────────────────────────────────────┘    │
└─────────────────────────┬───────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────┐
│                   Domain Layer                       │
│        (Entities, Value Objects, Domain Services)    │
└─────────────────────────────────────────────────────┘
```

### Implementing Application Services
```php
<?php
namespace Application\Order;

final class OrderApplicationService {
    public function __construct(
        private OrderRepository $orders,
        private CustomerRepository $customers,
        private ProductRepository $products,
        private InventoryService $inventory,
        private EventDispatcher $events,
        private TransactionManager $transaction
    ) {}
    
    public function placeOrder(PlaceOrderRequest $request): OrderId {
        return $this->transaction->execute(function () use ($request) {
            // 1. Load domain objects
            $customer = $this->customers->findOrFail($request->customerId);
            
            // 2. Validate business rules
            if (!$customer->canPlaceOrders()) {
                throw new CustomerCannotPlaceOrders($customer->id());
            }
            
            // 3. Build order items
            $items = [];
            foreach ($request->items as $itemData) {
                $product = $this->products->findOrFail($itemData->productId);
                
                if (!$this->inventory->isAvailable($product->id(), $itemData->quantity)) {
                    throw new InsufficientInventory($product->id());
                }
                
                $items[] = [
                    'productId' => $product->id(),
                    'quantity' => new Quantity($itemData->quantity),
                    'unitPrice' => $product->price()
                ];
            }
            
            // 4. Execute domain logic
            $order = Order::place(
                $this->orders->nextIdentity(),
                $customer->id(),
                $items
            );
            
            // 5. Persist
            $this->orders->save($order);
            
            // 6. Dispatch events
            $this->events->dispatchAll($order->pullDomainEvents());
            
            return $order->id();
        });
    }
    
    public function cancelOrder(CancelOrderRequest $request): void {
        $this->transaction->execute(function () use ($request) {
            $order = $this->orders->findOrFail($request->orderId);
            
            $order->cancel(new CancellationReason($request->reason));
            
            $this->orders->save($order);
            $this->events->dispatchAll($order->pullDomainEvents());
        });
    }
}
```

### Data Transfer Objects (DTOs)
```php
<?php
namespace Application\Order\DTO;

final readonly class PlaceOrderRequest {
    public function __construct(
        public CustomerId $customerId,
        public Address $shippingAddress,
        /** @var OrderItemRequest[] */
        public array $items
    ) {}
    
    public static function fromArray(array $data): self {
        return new self(
            customerId: CustomerId::fromString($data['customer_id']),
            shippingAddress: Address::fromArray($data['shipping_address']),
            items: array_map(
                fn($item) => OrderItemRequest::fromArray($item),
                $data['items']
            )
        );
    }
}

final readonly class OrderItemRequest {
    public function __construct(
        public ProductId $productId,
        public int $quantity
    ) {}
    
    public static function fromArray(array $data): self {
        return new self(
            productId: ProductId::fromString($data['product_id']),
            quantity: (int) $data['quantity']
        );
    }
}

// Response DTO
final readonly class OrderResponse {
    public function __construct(
        public string $id,
        public string $status,
        public string $total,
        public array $items,
        public string $createdAt
    ) {}
    
    public static function fromOrder(Order $order): self {
        return new self(
            id: $order->id()->toString(),
            status: $order->status()->value,
            total: $order->total()->format(),
            items: array_map(
                fn($line) => [
                    'product_id' => $line->productId()->toString(),
                    'quantity' => $line->quantity()->value(),
                    'unit_price' => $line->unitPrice()->format(),
                    'subtotal' => $line->subtotal()->format()
                ],
                $order->lines()
            ),
            createdAt: $order->createdAt()->format('c')
        );
    }
}
```

### Application Service Guidelines
### 1. Thin Application Services

```php
<?php
// BAD: Business logic in application service
class OrderService {
    public function placeOrder($data): void {
        $order = new Order();
        // Business logic here - should be in domain!
        if (count($data['items']) > 100) {
            throw new TooManyItems();
        }
        foreach ($data['items'] as $item) {
            $order->total += $item['price'] * $item['qty'];
        }
    }
}

// GOOD: Delegate to domain
class OrderService {
    public function placeOrder(PlaceOrderRequest $request): void {
        // Application service just coordinates
        $order = Order::place(...);
        // Business rules are inside Order::place()
    }
}
```

### 2. One Use Case Per Method

```php
<?php
// Clear, focused methods
class OrderApplicationService {
    public function placeOrder(...): OrderId {}
    public function cancelOrder(...): void {}
    public function shipOrder(...): void {}
    public function addItemToOrder(...): void {}
}
```

### 3. Transaction Boundaries

```php
<?php
class OrderApplicationService {
    public function placeOrder(PlaceOrderRequest $request): OrderId {
        // One transaction per use case
        return $this->transaction->execute(function () use ($request) {
            // All changes committed together or rolled back
        });
    }
}
```

## Common Pitfalls

1. **Overcomplicating simple cases** - Not every part of the application needs application services. Apply it where complexity warrants the investment.
2. **Ignoring the ubiquitous language** - Naming classes and methods without input from domain experts leads to a model that does not reflect the business.
3. **Mixing infrastructure concerns** - Allowing framework dependencies to leak into the domain layer undermines the architectural benefits.

## Best Practices

1. **Start from the domain** - Model the business concepts first, then figure out persistence and infrastructure.
2. **Keep it simple** - Use the simplest pattern that solves the problem. Introduce complexity only when needed.
3. **Collaborate with domain experts** - The model should be shaped by business knowledge, not just technical preferences.

## Summary

- Application Services is a fundamental concept in Domain-Driven Design
- Proper implementation leads to more maintainable and expressive code
- Always align your implementation with the ubiquitous language
- Apply these patterns where business complexity justifies the investment

## Resources

- [Application Services in DDD](https://www.domainlanguage.com/) — Domain Language - DDD resources

---

> 📘 *This lesson is part of the [Domain-Driven Design with PHP](https://stanza.dev/courses/php-ddd) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*