---
source_course: "laravel-queues-jobs"
source_lesson: "laravel-queues-jobs-job-configuration"
---

# Job Configuration Options

Configure how jobs behave with properties and methods on the job class.

## Retry Configuration

```php
class ProcessPodcast implements ShouldQueue
{
    /**
     * Number of times to retry.
     */
    public int $tries = 3;

    /**
     * Maximum exceptions before failing.
     */
    public int $maxExceptions = 3;

    /**
     * Seconds to wait before retrying.
     */
    public int $backoff = 10;

    /**
     * Exponential backoff: 10s, 30s, 60s
     */
    public array $backoff = [10, 30, 60];
}
```

### Time-Based Retries

```php
class ProcessPodcast implements ShouldQueue
{
    /**
     * Retry until this time.
     */
    public function retryUntil(): DateTime
    {
        return now()->addHours(24);
    }
}
```

## Timeout Configuration

```php
class ProcessVideo implements ShouldQueue
{
    /**
     * Job timeout in seconds.
     */
    public int $timeout = 300;  // 5 minutes

    /**
     * Fail if timeout exceeded.
     */
    public bool $failOnTimeout = true;
}
```

## Unique Jobs

Prevent duplicate jobs:

```php
use Illuminate\Contracts\Queue\ShouldBeUnique;

class UpdateSearchIndex implements ShouldQueue, ShouldBeUnique
{
    public function __construct(
        public Product $product
    ) {}

    /**
     * Unique identifier for the job.
     */
    public function uniqueId(): string
    {
        return $this->product->id;
    }

    /**
     * Seconds until uniqueness lock expires.
     */
    public int $uniqueFor = 3600;  // 1 hour
}

// Only one UpdateSearchIndex for product 123 can be queued
UpdateSearchIndex::dispatch($product);
UpdateSearchIndex::dispatch($product);  // Ignored (duplicate)
```

### Unique Until Processing

```php
use Illuminate\Contracts\Queue\ShouldBeUniqueUntilProcessing;

class GenerateReport implements ShouldQueue, ShouldBeUniqueUntilProcessing
{
    // Allows another job to be queued once this one starts processing
}
```

## Rate Limiting Jobs

```php
use Illuminate\Queue\Middleware\RateLimited;
use Illuminate\Support\Facades\RateLimiter;

// In AppServiceProvider
RateLimiter::for('api-calls', function ($job) {
    return Limit::perMinute(60);
});

// In your job
class CallExternalApi implements ShouldQueue
{
    public function middleware(): array
    {
        return [new RateLimited('api-calls')];
    }
}
```

## Preventing Overlapping Jobs

```php
use Illuminate\Queue\Middleware\WithoutOverlapping;

class UpdateUserBalance implements ShouldQueue
{
    public function __construct(
        public User $user
    ) {}

    public function middleware(): array
    {
        return [
            new WithoutOverlapping($this->user->id),
        ];
    }
}
```

### Release on Overlap

```php
public function middleware(): array
{
    return [
        (new WithoutOverlapping($this->user->id))
            ->releaseAfter(60)  // Try again in 60 seconds
            ->expireAfter(180), // Lock expires in 3 minutes
    ];
}
```

## Job Middleware

Apply middleware to jobs:

```php
use Illuminate\Queue\Middleware\ThrottlesExceptions;

class ProcessWebhook implements ShouldQueue
{
    public function middleware(): array
    {
        return [
            new ThrottlesExceptions(10, 5),  // 10 exceptions per 5 minutes
        ];
    }
}
```

### Custom Middleware

```php
class LogJobMiddleware
{
    public function handle($job, $next)
    {
        Log::info('Starting job', ['class' => get_class($job)]);

        $result = $next($job);

        Log::info('Finished job', ['class' => get_class($job)]);

        return $result;
    }
}

// In your job
public function middleware(): array
{
    return [new LogJobMiddleware];
}
```

## Resources

- [Job Middleware](https://laravel.com/docs/12.x/queues#job-middleware) — Official documentation on job middleware

---

> 📘 *This lesson is part of the [Laravel Background Processing](https://stanza.dev/courses/laravel-queues-jobs) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*