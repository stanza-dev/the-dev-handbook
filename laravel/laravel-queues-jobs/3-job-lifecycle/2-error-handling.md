---
source_course: "laravel-queues-jobs"
source_lesson: "laravel-queues-jobs-error-handling"
---

# Handling Failed Jobs

When jobs fail, Laravel provides tools to handle, retry, and debug them.

## The failed() Method

Define what happens when a job fails:

```php
class ProcessOrder implements ShouldQueue
{
    public function handle(): void
    {
        // Process order...
    }

    /**
     * Handle job failure.
     */
    public function failed(?Throwable $exception): void
    {
        // Log the failure
        Log::error('Order processing failed', [
            'order_id' => $this->order->id,
            'exception' => $exception->getMessage(),
        ]);

        // Notify someone
        Notification::send(
            User::admins()->get(),
            new JobFailed($this->order, $exception)
        );

        // Update status
        $this->order->update(['status' => 'failed']);
    }
}
```

## Manual Failure

```php
public function handle(): void
{
    if ($this->order->cancelled) {
        $this->fail('Order was cancelled.');
        return;
    }

    // Or fail with exception
    $this->fail(new OrderCancelledException());
}
```

## Deleting Jobs

```php
public function handle(): void
{
    if ($this->order->cancelled) {
        $this->delete();  // Remove from queue without failing
        return;
    }

    // Process...
}
```

## Releasing Jobs Back to Queue

```php
public function handle(): void
{
    if ($this->serviceUnavailable()) {
        $this->release(60);  // Try again in 60 seconds
        return;
    }

    // Process...
}
```

## Global Failed Job Handler

In `AppServiceProvider`:

```php
use Illuminate\Support\Facades\Queue;

public function boot(): void
{
    Queue::failing(function (JobFailed $event) {
        // $event->connectionName
        // $event->job
        // $event->exception

        Log::channel('slack')->error('Job failed', [
            'job' => $event->job->resolveName(),
            'exception' => $event->exception->getMessage(),
        ]);
    });
}
```

## Managing Failed Jobs

```bash
# List failed jobs
php artisan queue:failed

# Retry specific job
php artisan queue:retry ce7bb17c-cdd8-41f0-a8ec-7b4fef4e5ece

# Retry all failed jobs
php artisan queue:retry all

# Retry jobs that failed in last 24 hours
php artisan queue:retry --range=24

# Delete a failed job
php artisan queue:forget ce7bb17c-cdd8-41f0-a8ec-7b4fef4e5ece

# Clear all failed jobs
php artisan queue:flush

# Prune old failed jobs
php artisan queue:prune-failed --hours=48
```

## Ignoring Missing Models

```php
use Illuminate\Queue\Middleware\SkipIfModelMissing;

class SendWelcomeEmail implements ShouldQueue
{
    use SerializesModels;

    public function __construct(
        public User $user
    ) {}

    public function middleware(): array
    {
        return [new SkipIfModelMissing];
    }
}

// Or on the model property
class SendWelcomeEmail implements ShouldQueue
{
    public $deleteWhenMissingModels = true;
}
```

## Retry After Transaction

Ensure database transactions complete:

```php
class ProcessPayment implements ShouldQueue
{
    public $afterCommit = true;
}

// Or when dispatching
ProcessPayment::dispatch($payment)->afterCommit();
```

## Example: Robust Job with Full Error Handling

```php
<?php

namespace App\Jobs;

use App\Models\Order;
use App\Services\PaymentGateway;
use App\Notifications\PaymentFailed;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;
use Illuminate\Queue\Middleware\WithoutOverlapping;

class ProcessPayment implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    public int $tries = 3;
    public array $backoff = [30, 60, 120];
    public int $timeout = 60;
    public bool $failOnTimeout = true;
    public bool $deleteWhenMissingModels = true;

    public function __construct(
        public Order $order
    ) {}

    public function middleware(): array
    {
        return [
            new WithoutOverlapping($this->order->id),
        ];
    }

    public function handle(PaymentGateway $gateway): void
    {
        if ($this->order->isPaid()) {
            $this->delete();
            return;
        }

        try {
            $charge = $gateway->charge($this->order);
            $this->order->markAsPaid($charge->id);
        } catch (PaymentDeclinedException $e) {
            $this->fail($e);
        } catch (GatewayUnavailableException $e) {
            $this->release(60);
        }
    }

    public function failed(?Throwable $exception): void
    {
        $this->order->update(['status' => 'payment_failed']);
        $this->order->user->notify(new PaymentFailed($this->order, $exception));
    }
}
```

## Resources

- [Dealing with Failed Jobs](https://laravel.com/docs/12.x/queues#dealing-with-failed-jobs) — Official documentation on failed jobs

---

> 📘 *This lesson is part of the [Laravel Background Processing](https://stanza.dev/courses/laravel-queues-jobs) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*