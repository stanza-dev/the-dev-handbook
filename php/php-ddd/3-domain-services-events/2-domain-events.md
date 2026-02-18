---
source_course: "php-ddd"
source_lesson: "php-ddd-domain-events"
---

# Domain Events

**Domain Events** capture something important that happened in the domain. They're named in past tense and represent facts that have occurred.

## Why Domain Events?

1. **Decouple aggregates** - communicate without direct references
2. **Audit trail** - record what happened and when
3. **Trigger side effects** - notifications, integrations
4. **Enable eventual consistency** - coordinate across boundaries

## Implementing Domain Events

```php
<?php
namespace Domain\Shared;

interface DomainEvent {
    public function occurredAt(): \DateTimeImmutable;
    public function aggregateId(): string;
}

abstract class BaseDomainEvent implements DomainEvent {
    private \DateTimeImmutable $occurredAt;
    
    public function __construct() {
        $this->occurredAt = new \DateTimeImmutable();
    }
    
    public function occurredAt(): \DateTimeImmutable {
        return $this->occurredAt;
    }
}
```

### Concrete Events

```php
<?php
namespace Domain\Ordering\Events;

final class OrderWasPlaced extends BaseDomainEvent {
    public function __construct(
        public readonly OrderId $orderId,
        public readonly CustomerId $customerId,
        public readonly Money $total,
        public readonly array $lineItems
    ) {
        parent::__construct();
    }
    
    public function aggregateId(): string {
        return $this->orderId->toString();
    }
}

final class OrderWasShipped extends BaseDomainEvent {
    public function __construct(
        public readonly OrderId $orderId,
        public readonly ShipmentId $shipmentId,
        public readonly string $trackingNumber
    ) {
        parent::__construct();
    }
    
    public function aggregateId(): string {
        return $this->orderId->toString();
    }
}

final class OrderWasCancelled extends BaseDomainEvent {
    public function __construct(
        public readonly OrderId $orderId,
        public readonly CancellationReason $reason
    ) {
        parent::__construct();
    }
    
    public function aggregateId(): string {
        return $this->orderId->toString();
    }
}
```

## Recording Events in Aggregates

```php
<?php
trait RecordsEvents {
    private array $domainEvents = [];
    
    protected function recordThat(DomainEvent $event): void {
        $this->domainEvents[] = $event;
    }
    
    public function pullDomainEvents(): array {
        $events = $this->domainEvents;
        $this->domainEvents = [];
        return $events;
    }
}

final class Order {
    use RecordsEvents;
    
    public static function place(
        OrderId $id,
        CustomerId $customerId,
        array $items
    ): self {
        $order = new self($id, $customerId);
        
        foreach ($items as $item) {
            $order->addLine($item);
        }
        
        $order->recordThat(new OrderWasPlaced(
            orderId: $id,
            customerId: $customerId,
            total: $order->total(),
            lineItems: $order->lines()
        ));
        
        return $order;
    }
    
    public function cancel(CancellationReason $reason): void {
        if (!$this->status->canBeCancelled()) {
            throw CannotCancelOrder::withStatus($this->status);
        }
        
        $this->status = OrderStatus::Cancelled;
        
        $this->recordThat(new OrderWasCancelled(
            orderId: $this->id,
            reason: $reason
        ));
    }
}
```

## Event Handlers

```php
<?php
namespace Application\Ordering\Handlers;

final class SendOrderConfirmationEmail {
    public function __construct(
        private EmailService $emailService,
        private CustomerRepository $customers
    ) {}
    
    public function __invoke(OrderWasPlaced $event): void {
        $customer = $this->customers->find($event->customerId);
        
        $this->emailService->send(
            to: $customer->email(),
            template: 'order-confirmation',
            data: [
                'orderId' => $event->orderId->toString(),
                'total' => $event->total->format(),
                'items' => $event->lineItems
            ]
        );
    }
}

final class UpdateInventoryOnOrderPlaced {
    public function __construct(
        private InventoryService $inventory
    ) {}
    
    public function __invoke(OrderWasPlaced $event): void {
        foreach ($event->lineItems as $item) {
            $this->inventory->reserve(
                $item->productId,
                $item->quantity
            );
        }
    }
}

final class ReleaseInventoryOnOrderCancelled {
    public function __construct(
        private InventoryService $inventory,
        private OrderRepository $orders
    ) {}
    
    public function __invoke(OrderWasCancelled $event): void {
        $order = $this->orders->find($event->orderId);
        
        foreach ($order->lines() as $line) {
            $this->inventory->release(
                $line->productId(),
                $line->quantity()
            );
        }
    }
}
```

## Event Dispatcher

```php
<?php
namespace Infrastructure\Events;

final class EventDispatcher {
    /** @var array<class-string, callable[]> */
    private array $handlers = [];
    
    public function subscribe(string $eventClass, callable $handler): void {
        $this->handlers[$eventClass][] = $handler;
    }
    
    public function dispatch(object $event): void {
        $eventClass = get_class($event);
        
        foreach ($this->handlers[$eventClass] ?? [] as $handler) {
            $handler($event);
        }
    }
    
    public function dispatchAll(array $events): void {
        foreach ($events as $event) {
            $this->dispatch($event);
        }
    }
}

// Usage in Application Service
final class PlaceOrderHandler {
    public function handle(PlaceOrderCommand $command): OrderId {
        $order = Order::place(
            OrderId::generate(),
            $command->customerId,
            $command->items
        );
        
        $this->orderRepository->save($order);
        
        // Dispatch domain events
        $this->eventDispatcher->dispatchAll(
            $order->pullDomainEvents()
        );
        
        return $order->id();
    }
}
```

## Resources

- [Domain Events Pattern](https://martinfowler.com/eaaDev/DomainEvent.html) — Martin Fowler on Domain Events

---

> 📘 *This lesson is part of the [Domain-Driven Design with PHP](https://stanza.dev/courses/php-ddd) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*