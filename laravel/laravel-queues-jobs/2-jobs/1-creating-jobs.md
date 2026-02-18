---
source_course: "laravel-queues-jobs"
source_lesson: "laravel-queues-jobs-creating-jobs"
---

# Creating Job Classes

Jobs are classes that encapsulate work to be done in the background. Each job should do one specific task.

## Generating Jobs

```bash
php artisan make:job ProcessPodcast
```

This creates `app/Jobs/ProcessPodcast.php`:

```php
<?php

namespace App\Jobs;

use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;

class ProcessPodcast implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    /**
     * Create a new job instance.
     */
    public function __construct(
        public Podcast $podcast
    ) {}

    /**
     * Execute the job.
     */
    public function handle(): void
    {
        // Process the podcast...
    }
}
```

## Job Anatomy

### Traits Explained

| Trait | Purpose |
|-------|----------|
| `Dispatchable` | Enables `::dispatch()` method |
| `InteractsWithQueue` | Access to queue methods (`$this->delete()`, etc.) |
| `Queueable` | Queue, connection, and delay configuration |
| `SerializesModels` | Eloquent models are serialized and reloaded |

### ShouldQueue Interface

```php
// With ShouldQueue: Job is queued
class SendEmail implements ShouldQueue { }

// Without ShouldQueue: Job runs immediately (synchronously)
class SendEmail { }
```

## Passing Data to Jobs

```php
<?php

namespace App\Jobs;

use App\Models\User;
use App\Models\Report;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;

class GenerateReport implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    public function __construct(
        public User $user,
        public string $reportType,
        public array $options = []
    ) {}

    public function handle(): void
    {
        $report = Report::generate($this->reportType, $this->options);

        $this->user->notify(new ReportReady($report));
    }
}
```

### Model Serialization

Eloquent models are serialized by ID and reloaded:

```php
// When dispatching
GenerateReport::dispatch($user, 'sales');
// User is serialized as: {"class": "App\\Models\\User", "id": 123}

// When processing
// User is fetched fresh: User::find(123)
```

**Warning**: If the model is deleted before processing, the job will fail.

## The handle() Method

The `handle()` method contains your job logic:

```php
public function handle(
    AudioProcessor $processor,  // Dependency injection works!
    Storage $storage
): void {
    $processedFile = $processor->process($this->podcast->audio_path);

    $storage->put(
        "podcasts/{$this->podcast->id}/processed.mp3",
        $processedFile
    );

    $this->podcast->update(['processed' => true]);
}
```

## Complete Job Example

```php
<?php

namespace App\Jobs;

use App\Models\Order;
use App\Services\PaymentGateway;
use App\Notifications\OrderConfirmation;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;
use Illuminate\Support\Facades\Log;

class ProcessOrder implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    /**
     * Number of retry attempts.
     */
    public int $tries = 3;

    /**
     * Timeout in seconds.
     */
    public int $timeout = 120;

    public function __construct(
        public Order $order
    ) {}

    public function handle(PaymentGateway $gateway): void
    {
        Log::info('Processing order', ['order_id' => $this->order->id]);

        // Charge the customer
        $charge = $gateway->charge(
            $this->order->user,
            $this->order->total
        );

        // Update order status
        $this->order->update([
            'status' => 'paid',
            'charge_id' => $charge->id,
        ]);

        // Send confirmation
        $this->order->user->notify(new OrderConfirmation($this->order));
    }

    /**
     * Handle job failure.
     */
    public function failed(?Throwable $exception): void
    {
        Log::error('Order processing failed', [
            'order_id' => $this->order->id,
            'error' => $exception->getMessage(),
        ]);

        $this->order->update(['status' => 'failed']);
    }
}
```

## Resources

- [Creating Jobs](https://laravel.com/docs/12.x/queues#creating-jobs) — Official documentation on creating jobs

---

> 📘 *This lesson is part of the [Laravel Background Processing](https://stanza.dev/courses/laravel-queues-jobs) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*