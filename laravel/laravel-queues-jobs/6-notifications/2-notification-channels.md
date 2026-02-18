---
source_course: "laravel-queues-jobs"
source_lesson: "laravel-queues-jobs-notification-channels"
---

# Custom Notification Channels

Laravel supports many notification channels beyond email and database. Let's explore Slack, SMS, and creating custom channels.

## Slack Notifications

```bash
composer require laravel/slack-notification-channel
```

```php
use Illuminate\Notifications\Messages\SlackMessage;

public function via(object $notifiable): array
{
    return ['slack'];
}

public function toSlack(object $notifiable): SlackMessage
{
    return (new SlackMessage)
        ->success()  // or ->warning(), ->error()
        ->content('Invoice Paid!')
        ->attachment(function ($attachment) {
            $attachment->title('Invoice #' . $this->invoice->number)
                ->fields([
                    'Amount' => '$' . $this->invoice->amount,
                    'Customer' => $this->invoice->customer->name,
                ]);
        });
}
```

Configure Slack webhook:

```php
// In User model or as on-demand route
public function routeNotificationForSlack(): string
{
    return 'https://hooks.slack.com/services/...';
}
```

## SMS Notifications (Vonage)

```bash
composer require laravel/vonage-notification-channel
```

```php
use Illuminate\Notifications\Messages\VonageMessage;

public function via(object $notifiable): array
{
    return ['vonage'];
}

public function toVonage(object $notifiable): VonageMessage
{
    return (new VonageMessage)
        ->content('Your order has shipped! Track: ' . $this->order->tracking);
}

// In User model
public function routeNotificationForVonage(): string
{
    return $this->phone_number;
}
```

## Broadcast Notifications

For real-time notifications via WebSockets:

```php
use Illuminate\Notifications\Messages\BroadcastMessage;

public function via(object $notifiable): array
{
    return ['broadcast', 'database'];
}

public function toBroadcast(object $notifiable): BroadcastMessage
{
    return new BroadcastMessage([
        'invoice_id' => $this->invoice->id,
        'amount' => $this->invoice->amount,
        'message' => 'New invoice paid!',
    ]);
}
```

Listen in JavaScript:

```javascript
Echo.private(`App.Models.User.${userId}`)
    .notification((notification) => {
        console.log(notification.message);
    });
```

## Creating Custom Channels

```php
<?php

namespace App\Channels;

use Illuminate\Notifications\Notification;

class TelegramChannel
{
    public function send($notifiable, Notification $notification)
    {
        $message = $notification->toTelegram($notifiable);

        $chatId = $notifiable->routeNotificationFor('telegram');

        // Send to Telegram API
        Http::post('https://api.telegram.org/bot'.config('services.telegram.token').'/sendMessage', [
            'chat_id' => $chatId,
            'text' => $message->content,
        ]);
    }
}
```

Use in notification:

```php
use App\Channels\TelegramChannel;

public function via(object $notifiable): array
{
    return [TelegramChannel::class];
}

public function toTelegram(object $notifiable)
{
    return new TelegramMessage('Your order has shipped!');
}
```

## Queued Notifications

```php
class InvoicePaid extends Notification implements ShouldQueue
{
    use Queueable;

    // Configure queue
    public $connection = 'redis';
    public $queue = 'notifications';
    public $delay = 60;  // seconds

    // Conditional queueing
    public function shouldSend(object $notifiable, string $channel): bool
    {
        return $notifiable->wants_notifications;
    }
}
```

## Notification Events

Listen for notification events:

```php
use Illuminate\Notifications\Events\NotificationSent;
use Illuminate\Notifications\Events\NotificationFailed;

// In AppServiceProvider
Event::listen(NotificationSent::class, function ($event) {
    Log::info('Notification sent', [
        'notifiable' => $event->notifiable,
        'notification' => $event->notification,
        'channel' => $event->channel,
    ]);
});

Event::listen(NotificationFailed::class, function ($event) {
    Log::error('Notification failed', [
        'exception' => $event->exception,
    ]);
});
```

## Complete Multi-Channel Example

```php
<?php

namespace App\Notifications;

use App\Models\Order;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Notifications\Messages\MailMessage;
use Illuminate\Notifications\Messages\SlackMessage;
use Illuminate\Notifications\Notification;

class OrderShipped extends Notification implements ShouldQueue
{
    use Queueable;

    public function __construct(
        public Order $order
    ) {}

    public function via(object $notifiable): array
    {
        $channels = ['mail', 'database'];

        if ($notifiable->slack_webhook_url) {
            $channels[] = 'slack';
        }

        if ($notifiable->phone && $this->order->total > 100) {
            $channels[] = 'vonage';
        }

        return $channels;
    }

    public function toMail(object $notifiable): MailMessage
    {
        return (new MailMessage)
            ->subject('Your Order Has Shipped!')
            ->greeting('Great news, ' . $notifiable->name . '!')
            ->line('Your order #' . $this->order->number . ' has been shipped.')
            ->line('Tracking: ' . $this->order->tracking_number)
            ->action('Track Order', url('/orders/' . $this->order->id))
            ->line('Thank you for shopping with us!');
    }

    public function toSlack(object $notifiable): SlackMessage
    {
        return (new SlackMessage)
            ->success()
            ->content('Order shipped!')
            ->attachment(function ($attachment) {
                $attachment->title('Order #' . $this->order->number)
                    ->fields([
                        'Customer' => $this->order->user->name,
                        'Total' => '$' . $this->order->total,
                        'Tracking' => $this->order->tracking_number,
                    ]);
            });
    }

    public function toArray(object $notifiable): array
    {
        return [
            'order_id' => $this->order->id,
            'message' => 'Order #' . $this->order->number . ' shipped!',
            'tracking' => $this->order->tracking_number,
        ];
    }
}
```

## Resources

- [Notification Channels](https://laravel.com/docs/12.x/notifications#specifying-delivery-channels) — Documentation on notification channels

---

> 📘 *This lesson is part of the [Laravel Background Processing](https://stanza.dev/courses/laravel-queues-jobs) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*