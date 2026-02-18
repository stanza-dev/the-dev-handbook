---
source_course: "laravel-queues-jobs"
source_lesson: "laravel-queues-jobs-events-introduction"
---

# Introduction to Events

Events provide a simple observer pattern implementation, allowing you to subscribe to and respond to various events in your application.

## Why Events?

```
Without Events:                     With Events:
┌───────────────────────┐           ┌───────────────────────┐
│   OrderController     │           │   OrderController     │
│                       │           │                       │
│ - Process payment     │           │ - Process payment     │
│ - Send confirmation   │           │ - Dispatch OrderPlaced│
│ - Update inventory    │           └───────────────────────┘
│ - Notify warehouse    │                      │
│ - Add to analytics    │                      ▼
│ - Send to CRM         │           ┌───────────────────────┐
│   (tightly coupled!)  │           │   Event: OrderPlaced  │
└───────────────────────┘           └───────────────────────┘
                                           │
                        ┌──────────────────┼──────────────────┐
                        ▼                  ▼                  ▼
                 ┌─────────────┐   ┌─────────────┐   ┌─────────────┐
                 │SendEmailJob │   │UpdateStock  │   │Analytics    │
                 └─────────────┘   └─────────────┘   └─────────────┘
```

## Creating Events

```bash
php artisan make:event OrderPlaced
```

This creates `app/Events/OrderPlaced.php`:

```php
<?php

namespace App\Events;

use App\Models\Order;
use Illuminate\Broadcasting\InteractsWithSockets;
use Illuminate\Foundation\Events\Dispatchable;
use Illuminate\Queue\SerializesModels;

class OrderPlaced
{
    use Dispatchable, InteractsWithSockets, SerializesModels;

    public function __construct(
        public Order $order
    ) {}
}
```

## Creating Listeners

```bash
php artisan make:listener SendOrderConfirmation --event=OrderPlaced
```

This creates `app/Listeners/SendOrderConfirmation.php`:

```php
<?php

namespace App\Listeners;

use App\Events\OrderPlaced;
use App\Notifications\OrderConfirmation;

class SendOrderConfirmation
{
    public function handle(OrderPlaced $event): void
    {
        $event->order->user->notify(
            new OrderConfirmation($event->order)
        );
    }
}
```

## Registering Events & Listeners

In `AppServiceProvider`:

```php
use App\Events\OrderPlaced;
use App\Listeners\SendOrderConfirmation;
use App\Listeners\UpdateInventory;
use Illuminate\Support\Facades\Event;

public function boot(): void
{
    Event::listen(
        OrderPlaced::class,
        SendOrderConfirmation::class
    );

    Event::listen(
        OrderPlaced::class,
        UpdateInventory::class
    );
}
```

### Using Closures

```php
Event::listen(function (OrderPlaced $event) {
    Log::info('Order placed', ['order_id' => $event->order->id]);
});
```

## Dispatching Events

```php
use App\Events\OrderPlaced;

class OrderController extends Controller
{
    public function store(Request $request)
    {
        $order = Order::create($request->validated());

        // Dispatch the event
        OrderPlaced::dispatch($order);

        // Or using event() helper
        event(new OrderPlaced($order));

        return redirect()->route('orders.show', $order);
    }
}
```

## Queued Listeners

Process listeners in the background:

```php
<?php

namespace App\Listeners;

use App\Events\OrderPlaced;
use Illuminate\Contracts\Queue\ShouldQueue;

class UpdateInventory implements ShouldQueue
{
    public function handle(OrderPlaced $event): void
    {
        foreach ($event->order->items as $item) {
            $item->product->decrement('stock', $item->quantity);
        }
    }
}
```

### Queued Listener Configuration

```php
class UpdateInventory implements ShouldQueue
{
    public $connection = 'redis';
    public $queue = 'inventory';
    public $delay = 10;  // seconds

    public function viaQueue(): string
    {
        return 'inventory';
    }

    public function shouldQueue(OrderPlaced $event): bool
    {
        return $event->order->total > 100;
    }
}
```

## Event Subscribers

Group related listeners:

```php
<?php

namespace App\Listeners;

use App\Events\OrderPlaced;
use App\Events\OrderShipped;
use App\Events\OrderCancelled;
use Illuminate\Events\Dispatcher;

class OrderEventSubscriber
{
    public function handleOrderPlaced(OrderPlaced $event): void
    {
        // Handle order placed
    }

    public function handleOrderShipped(OrderShipped $event): void
    {
        // Handle order shipped
    }

    public function subscribe(Dispatcher $events): void
    {
        $events->listen(
            OrderPlaced::class,
            [self::class, 'handleOrderPlaced']
        );

        $events->listen(
            OrderShipped::class,
            [self::class, 'handleOrderShipped']
        );
    }
}
```

Register in `AppServiceProvider`:

```php
Event::subscribe(OrderEventSubscriber::class);
```

## Resources

- [Events](https://laravel.com/docs/12.x/events) — Official Laravel events documentation

---

> 📘 *This lesson is part of the [Laravel Background Processing](https://stanza.dev/courses/laravel-queues-jobs) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*