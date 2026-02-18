---
source_course: "php-ddd"
source_lesson: "php-ddd-ports-adapters"
---

# Ports and Adapters (Hexagonal Architecture)

**Ports and Adapters** is another way to visualize Clean Architecture. The application core defines "ports" (interfaces), and "adapters" connect the outside world.

## The Hexagonal Model

```
                    ┌─────────────────┐
          REST API  │    Adapter      │  CLI
              ◄─────┤   (Primary/     ├─────►
                    │    Driving)     │
                    └────────┬────────┘
                             │
                    ┌────────▼────────┐
                    │                 │
           Port ────┤   Application   ├──── Port
         (Input)    │      Core       │   (Output)
                    │                 │
                    └────────┬────────┘
                             │
                    ┌────────▼────────┐
         Database   │    Adapter      │  External API
              ◄─────┤  (Secondary/    ├─────►
                    │   Driven)       │
                    └─────────────────┘
```

## Primary Ports (Driving/Input)

Ports that drive the application - user actions:

```php
<?php
namespace Application\Port\Input;

// Port: What the application can do
interface PlaceOrder {
    public function execute(PlaceOrderRequest $request): OrderId;
}

interface CancelOrder {
    public function execute(CancelOrderRequest $request): void;
}

interface GetOrder {
    public function execute(string $orderId): ?OrderResponse;
}
```

## Primary Adapters (Controllers)

```php
<?php
namespace Infrastructure\Adapter\Input\Http;

use Application\Port\Input\PlaceOrder;

// Adapter: HTTP to Application
final class OrderController {
    public function __construct(
        private PlaceOrder $placeOrder,
        private GetOrder $getOrder
    ) {}
    
    public function place(Request $request): Response {
        $orderId = $this->placeOrder->execute(
            new PlaceOrderRequest(
                customerId: $request->json('customer_id'),
                items: $request->json('items')
            )
        );
        
        return new JsonResponse(['id' => $orderId->toString()], 201);
    }
}

namespace Infrastructure\Adapter\Input\Cli;

// Adapter: CLI to Application
final class PlaceOrderCommand extends Command {
    protected function execute(InputInterface $input): int {
        $orderId = $this->placeOrder->execute(
            new PlaceOrderRequest(
                customerId: $input->getArgument('customer'),
                items: json_decode($input->getArgument('items'), true)
            )
        );
        
        $this->output->writeln("Order created: {$orderId}");
        return 0;
    }
}
```

## Secondary Ports (Driven/Output)

Ports the application uses - infrastructure needs:

```php
<?php
namespace Application\Port\Output;

// What the application needs from outside
interface OrderPersistence {
    public function save(Order $order): void;
    public function find(OrderId $id): ?Order;
    public function nextIdentity(): OrderId;
}

interface PaymentProcessor {
    public function charge(Money $amount, PaymentMethod $method): PaymentResult;
    public function refund(PaymentId $paymentId): RefundResult;
}

interface NotificationSender {
    public function sendOrderConfirmation(Order $order, Customer $customer): void;
    public function sendShipmentNotification(Shipment $shipment): void;
}

interface InventoryChecker {
    public function isAvailable(ProductId $productId, Quantity $quantity): bool;
    public function reserve(ProductId $productId, Quantity $quantity): Reservation;
}
```

## Secondary Adapters (Infrastructure)

```php
<?php
namespace Infrastructure\Adapter\Output\Persistence;

use Application\Port\Output\OrderPersistence;

// Adapter: Application to Database
final class DoctrineOrderPersistence implements OrderPersistence {
    public function __construct(
        private EntityManagerInterface $em
    ) {}
    
    public function save(Order $order): void {
        $this->em->persist($order);
        $this->em->flush();
    }
    
    public function find(OrderId $id): ?Order {
        return $this->em->find(Order::class, $id->toString());
    }
}

namespace Infrastructure\Adapter\Output\Payment;

use Application\Port\Output\PaymentProcessor;

// Adapter: Application to Stripe
final class StripePaymentProcessor implements PaymentProcessor {
    public function __construct(
        private StripeClient $stripe
    ) {}
    
    public function charge(Money $amount, PaymentMethod $method): PaymentResult {
        try {
            $charge = $this->stripe->charges->create([
                'amount' => $amount->cents(),
                'currency' => strtolower($amount->currency()->value),
                'source' => $method->token(),
            ]);
            
            return PaymentResult::success(
                PaymentId::fromString($charge->id)
            );
        } catch (StripeException $e) {
            return PaymentResult::failed($e->getMessage());
        }
    }
}
```

## Testing with Ports and Adapters

```php
<?php
namespace Tests\Application;

class PlaceOrderTest extends TestCase {
    public function test_places_order_successfully(): void {
        // Use test adapters
        $persistence = new InMemoryOrderPersistence();
        $inventory = new AlwaysAvailableInventory();
        $events = new CollectingEventDispatcher();
        
        $useCase = new PlaceOrderUseCase(
            $persistence,
            $inventory,
            $events
        );
        
        $orderId = $useCase->execute(new PlaceOrderRequest(
            customerId: 'customer-1',
            items: [['product_id' => 'prod-1', 'quantity' => 2]]
        ));
        
        // Assert
        $this->assertNotNull($persistence->find($orderId));
        $this->assertCount(1, $events->dispatched());
    }
}

// Test doubles
final class InMemoryOrderPersistence implements OrderPersistence {
    private array $orders = [];
    
    public function save(Order $order): void {
        $this->orders[$order->id()->toString()] = $order;
    }
    
    public function find(OrderId $id): ?Order {
        return $this->orders[$id->toString()] ?? null;
    }
}

final class AlwaysAvailableInventory implements InventoryChecker {
    public function isAvailable(ProductId $p, Quantity $q): bool {
        return true;
    }
}
```

## Resources

- [Hexagonal Architecture](https://alistair.cockburn.us/hexagonal-architecture/) — Original Hexagonal Architecture article by Alistair Cockburn

---

> 📘 *This lesson is part of the [Domain-Driven Design with PHP](https://stanza.dev/courses/php-ddd) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*