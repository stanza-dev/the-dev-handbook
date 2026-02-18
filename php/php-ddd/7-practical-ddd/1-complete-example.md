---
source_course: "php-ddd"
source_lesson: "php-ddd-complete-example"
---

# Complete DDD Example: Order Processing

Let's build a complete, production-ready order processing system using all DDD patterns we've learned.

## Domain Model

```php
<?php
declare(strict_types=1);

namespace Domain\Ordering;

final class Order {
    use RecordsDomainEvents;
    
    private OrderId $id;
    private CustomerId $customerId;
    private OrderStatus $status;
    private Money $subtotal;
    private Money $tax;
    private Money $total;
    /** @var array<string, OrderLine> */
    private array $lines = [];
    private ?ShippingAddress $shippingAddress = null;
    private \DateTimeImmutable $createdAt;
    private ?\DateTimeImmutable $submittedAt = null;
    
    private function __construct(OrderId $id, CustomerId $customerId) {
        $this->id = $id;
        $this->customerId = $customerId;
        $this->status = OrderStatus::Draft;
        $this->subtotal = Money::zero();
        $this->tax = Money::zero();
        $this->total = Money::zero();
        $this->createdAt = new \DateTimeImmutable();
    }
    
    public static function create(OrderId $id, CustomerId $customerId): self {
        $order = new self($id, $customerId);
        $order->recordThat(new OrderCreated($id, $customerId));
        return $order;
    }
    
    public function addProduct(
        ProductId $productId,
        ProductName $name,
        Quantity $quantity,
        Money $unitPrice
    ): void {
        $this->assertDraft();
        
        $key = $productId->toString();
        
        if (isset($this->lines[$key])) {
            $this->lines[$key] = $this->lines[$key]->increaseQuantity($quantity);
        } else {
            $this->lines[$key] = new OrderLine(
                productId: $productId,
                productName: $name,
                quantity: $quantity,
                unitPrice: $unitPrice
            );
        }
        
        $this->recalculateTotals();
        $this->recordThat(new ProductAddedToOrder($this->id, $productId, $quantity));
    }
    
    public function removeProduct(ProductId $productId): void {
        $this->assertDraft();
        
        $key = $productId->toString();
        if (!isset($this->lines[$key])) {
            throw new ProductNotInOrder($productId);
        }
        
        unset($this->lines[$key]);
        $this->recalculateTotals();
    }
    
    public function setShippingAddress(ShippingAddress $address): void {
        $this->assertDraft();
        $this->shippingAddress = $address;
    }
    
    public function submit(): void {
        $this->assertDraft();
        
        if (empty($this->lines)) {
            throw OrderException::cannotSubmitEmpty();
        }
        
        if ($this->shippingAddress === null) {
            throw OrderException::shippingAddressRequired();
        }
        
        $this->status = OrderStatus::Submitted;
        $this->submittedAt = new \DateTimeImmutable();
        
        $this->recordThat(new OrderSubmitted(
            orderId: $this->id,
            customerId: $this->customerId,
            total: $this->total,
            lineCount: count($this->lines)
        ));
    }
    
    public function confirm(): void {
        if ($this->status !== OrderStatus::Submitted) {
            throw OrderException::cannotConfirm($this->status);
        }
        
        $this->status = OrderStatus::Confirmed;
        $this->recordThat(new OrderConfirmed($this->id));
    }
    
    public function cancel(CancellationReason $reason): void {
        if (!$this->status->canBeCancelled()) {
            throw OrderException::cannotCancel($this->status);
        }
        
        $this->status = OrderStatus::Cancelled;
        $this->recordThat(new OrderCancelled($this->id, $reason));
    }
    
    // Getters
    public function id(): OrderId { return $this->id; }
    public function customerId(): CustomerId { return $this->customerId; }
    public function status(): OrderStatus { return $this->status; }
    public function total(): Money { return $this->total; }
    /** @return OrderLine[] */
    public function lines(): array { return array_values($this->lines); }
    
    private function assertDraft(): void {
        if ($this->status !== OrderStatus::Draft) {
            throw OrderException::notModifiable($this->status);
        }
    }
    
    private function recalculateTotals(): void {
        $this->subtotal = array_reduce(
            $this->lines,
            fn(Money $sum, OrderLine $line) => $sum->add($line->lineTotal()),
            Money::zero()
        );
        
        // 10% tax rate
        $this->tax = $this->subtotal->multiply(0.10);
        $this->total = $this->subtotal->add($this->tax);
    }
}
```

## Application Service

```php
<?php
namespace Application\Ordering;

final class OrderApplicationService {
    public function __construct(
        private OrderRepository $orders,
        private CustomerRepository $customers,
        private ProductCatalog $catalog,
        private InventoryService $inventory,
        private PaymentService $payments,
        private EventDispatcher $events,
        private TransactionManager $tx
    ) {}
    
    public function createOrder(CreateOrderCommand $cmd): OrderId {
        return $this->tx->execute(function () use ($cmd) {
            $customer = $this->customers->findOrFail($cmd->customerId);
            
            $order = Order::create(
                $this->orders->nextIdentity(),
                $customer->id()
            );
            
            foreach ($cmd->items as $item) {
                $product = $this->catalog->findOrFail($item->productId);
                
                $order->addProduct(
                    $product->id(),
                    $product->name(),
                    new Quantity($item->quantity),
                    $product->price()
                );
            }
            
            if ($cmd->shippingAddress) {
                $order->setShippingAddress(
                    ShippingAddress::fromArray($cmd->shippingAddress)
                );
            }
            
            $this->orders->save($order);
            $this->events->dispatchAll($order->pullDomainEvents());
            
            return $order->id();
        });
    }
    
    public function submitOrder(SubmitOrderCommand $cmd): void {
        $this->tx->execute(function () use ($cmd) {
            $order = $this->orders->findOrFail($cmd->orderId);
            
            // Check inventory for all items
            foreach ($order->lines() as $line) {
                if (!$this->inventory->isAvailable($line->productId(), $line->quantity())) {
                    throw new InsufficientInventory($line->productId());
                }
            }
            
            // Reserve inventory
            foreach ($order->lines() as $line) {
                $this->inventory->reserve($line->productId(), $line->quantity());
            }
            
            $order->submit();
            
            $this->orders->save($order);
            $this->events->dispatchAll($order->pullDomainEvents());
        });
    }
    
    public function processPayment(ProcessPaymentCommand $cmd): void {
        $this->tx->execute(function () use ($cmd) {
            $order = $this->orders->findOrFail($cmd->orderId);
            
            $result = $this->payments->charge(
                $order->total(),
                $cmd->paymentMethod
            );
            
            if (!$result->isSuccessful()) {
                throw new PaymentFailed($result->errorMessage());
            }
            
            $order->confirm();
            
            $this->orders->save($order);
            $this->events->dispatchAll($order->pullDomainEvents());
        });
    }
}
```

## Event Handlers

```php
<?php
namespace Application\Ordering\EventHandler;

final class SendOrderConfirmationEmail {
    public function __construct(
        private CustomerRepository $customers,
        private OrderRepository $orders,
        private EmailSender $email
    ) {}
    
    public function __invoke(OrderConfirmed $event): void {
        $order = $this->orders->find($event->orderId);
        $customer = $this->customers->find($order->customerId());
        
        $this->email->send(
            to: $customer->email(),
            subject: "Order {$event->orderId} Confirmed",
            template: 'order-confirmed',
            data: ['order' => OrderView::fromOrder($order)]
        );
    }
}

final class UpdateAnalyticsOnOrderSubmitted {
    public function __construct(private AnalyticsService $analytics) {}
    
    public function __invoke(OrderSubmitted $event): void {
        $this->analytics->trackOrder(
            orderId: $event->orderId->toString(),
            customerId: $event->customerId->toString(),
            total: $event->total->cents(),
            itemCount: $event->lineCount
        );
    }
}
```

## Controller

```php
<?php
namespace Infrastructure\Http\Controller;

final class OrderController {
    public function __construct(
        private OrderApplicationService $orderService,
        private OrderQueryService $queries
    ) {}
    
    public function create(Request $request): Response {
        $command = new CreateOrderCommand(
            customerId: CustomerId::fromString($request->json('customer_id')),
            items: array_map(
                fn($i) => new OrderItemInput(
                    ProductId::fromString($i['product_id']),
                    $i['quantity']
                ),
                $request->json('items')
            ),
            shippingAddress: $request->json('shipping_address')
        );
        
        $orderId = $this->orderService->createOrder($command);
        
        return new JsonResponse(
            ['id' => $orderId->toString()],
            Response::HTTP_CREATED
        );
    }
    
    public function show(string $id): Response {
        $order = $this->queries->getOrder(new GetOrderQuery($id));
        
        if (!$order) {
            throw new NotFoundHttpException('Order not found');
        }
        
        return new JsonResponse($order);
    }
}
```

## Resources

- [Implementing Domain-Driven Design](https://www.oreilly.com/library/view/implementing-domain-driven-design/9780133039900/) — Vaughn Vernon's comprehensive DDD implementation guide

---

> 📘 *This lesson is part of the [Domain-Driven Design with PHP](https://stanza.dev/courses/php-ddd) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*