---
source_course: "laravel-queues-jobs"
source_lesson: "laravel-queues-jobs-notifications-introduction"
---

# Introduction to Notifications

Laravel's notification system provides a unified API to send notifications across various channels: email, SMS, Slack, database, and more.

## Creating Notifications

```bash
php artisan make:notification InvoicePaid
```

This creates `app/Notifications/InvoicePaid.php`:

```php
<?php

namespace App\Notifications;

use App\Models\Invoice;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Notifications\Messages\MailMessage;
use Illuminate\Notifications\Notification;

class InvoicePaid extends Notification implements ShouldQueue
{
    use Queueable;

    public function __construct(
        public Invoice $invoice
    ) {}

    /**
     * Notification channels.
     */
    public function via(object $notifiable): array
    {
        return ['mail', 'database'];
    }

    /**
     * Email representation.
     */
    public function toMail(object $notifiable): MailMessage
    {
        return (new MailMessage)
            ->subject('Invoice Paid')
            ->greeting('Hello!')
            ->line('Your invoice has been paid.')
            ->line('Amount: $' . number_format($this->invoice->amount, 2))
            ->action('View Invoice', url('/invoices/' . $this->invoice->id))
            ->line('Thank you for your business!');
    }

    /**
     * Database representation.
     */
    public function toArray(object $notifiable): array
    {
        return [
            'invoice_id' => $this->invoice->id,
            'amount' => $this->invoice->amount,
            'message' => 'Invoice #' . $this->invoice->number . ' was paid.',
        ];
    }
}
```

## Sending Notifications

### Using the Notifiable Trait

```php
use App\Notifications\InvoicePaid;

// Send to a user
$user->notify(new InvoicePaid($invoice));
```

### Using the Notification Facade

```php
use Illuminate\Support\Facades\Notification;

// Send to multiple users
Notification::send($users, new InvoicePaid($invoice));

// Send to anonymous recipient (for non-model notifications)
Notification::route('mail', 'customer@example.com')
    ->route('slack', 'https://hooks.slack.com/...')
    ->notify(new InvoicePaid($invoice));
```

## Notification Channels

### The via() Method

```php
public function via(object $notifiable): array
{
    // Always email
    return ['mail'];

    // Based on user preferences
    return $notifiable->notification_preferences;

    // Conditional channels
    $channels = ['database'];

    if ($notifiable->wants_email) {
        $channels[] = 'mail';
    }

    if ($this->invoice->amount > 1000) {
        $channels[] = 'slack';
    }

    return $channels;
}
```

## Mail Notifications

```php
public function toMail(object $notifiable): MailMessage
{
    return (new MailMessage)
        ->subject('Order Shipped')
        ->greeting('Hello ' . $notifiable->name . '!')
        ->line('Your order has been shipped.')
        ->line('Tracking Number: ' . $this->order->tracking_number)
        ->action('Track Order', url('/orders/' . $this->order->id))
        ->line('Thanks for shopping with us!')
        ->salutation('Best regards, The Team');
}

// With custom view
public function toMail(object $notifiable): MailMessage
{
    return (new MailMessage)
        ->subject('Your Invoice')
        ->view('emails.invoice', [
            'invoice' => $this->invoice,
            'user' => $notifiable,
        ]);
}

// Attachments
public function toMail(object $notifiable): MailMessage
{
    return (new MailMessage)
        ->subject('Monthly Report')
        ->line('Please find attached your monthly report.')
        ->attach('/path/to/report.pdf')
        ->attachData($this->pdf, 'report.pdf', [
            'mime' => 'application/pdf',
        ]);
}
```

## Database Notifications

```bash
php artisan make:notifications-table
php artisan migrate
```

Add trait to User model:

```php
use Illuminate\Notifications\Notifiable;

class User extends Authenticatable
{
    use Notifiable;
}
```

Retrieve notifications:

```php
// All notifications
$notifications = $user->notifications;

// Unread only
$unread = $user->unreadNotifications;

// Mark as read
$user->unreadNotifications->markAsRead();

// Mark single as read
$notification->markAsRead();

// Delete old notifications
$user->notifications()->where('created_at', '<', now()->subMonth())->delete();
```

In your views:

```blade
@foreach($user->unreadNotifications as $notification)
    <div class="notification">
        {{ $notification->data['message'] }}
        <small>{{ $notification->created_at->diffForHumans() }}</small>
    </div>
@endforeach
```

## Resources

- [Notifications](https://laravel.com/docs/12.x/notifications) — Official Laravel notifications documentation

---

> 📘 *This lesson is part of the [Laravel Background Processing](https://stanza.dev/courses/laravel-queues-jobs) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*