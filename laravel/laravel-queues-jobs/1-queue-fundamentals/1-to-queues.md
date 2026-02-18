---
source_course: "laravel-queues-jobs"
source_lesson: "laravel-queues-jobs-introduction-to-queues"
---

# Introduction to Queues

Queues allow you to defer time-consuming tasks—like sending emails or processing uploads—to be handled in the background. This keeps your web responses fast.

## Why Use Queues?

```
Without Queues:                 With Queues:
┌──────────────────┐            ┌──────────────────┐
│  User Request    │            │  User Request    │
└────────┬─────────┘            └────────┬─────────┘
         │                                │
         ▼                                ▼
┌──────────────────┐            ┌──────────────────┐
│  Process Order   │            │  Process Order   │
│   (100ms)        │            │   (100ms)        │
└────────┬─────────┘            └────────┬─────────┘
         │                                │
         ▼                                ▼
┌──────────────────┐            ┌──────────────────┐
│  Send Email      │            │  Queue Email Job │ ◄── Returns immediately!
│   (2000ms)       │            │   (5ms)          │
└────────┬─────────┘            └────────┬─────────┘
         │                                │
         ▼                                ▼
┌──────────────────┐            ┌──────────────────┐
│  Generate PDF    │            │  Response: 105ms │
│   (3000ms)       │            └──────────────────┘
└────────┬─────────┘
         │                      Background Worker:
         ▼                      ┌──────────────────┐
┌──────────────────┐            │  Send Email      │
│  Response: 5100ms│            │  Generate PDF    │
└──────────────────┘            │  (No user wait)  │
                                └──────────────────┘
```

## Queue Benefits

| Benefit | Description |
|---------|-------------|
| **Faster Responses** | Users don't wait for slow operations |
| **Better UX** | Immediate feedback, background processing |
| **Scalability** | Add more workers to handle load |
| **Reliability** | Retry failed jobs automatically |
| **Resource Management** | Control when heavy tasks run |

## Queue Drivers

Laravel supports multiple queue backends:

```php
// config/queue.php
'connections' => [
    'database' => [
        'driver' => 'database',
        'connection' => env('DB_QUEUE_CONNECTION'),
        'table' => env('DB_QUEUE_TABLE', 'jobs'),
        'queue' => env('DB_QUEUE', 'default'),
        'retry_after' => (int) env('DB_QUEUE_RETRY_AFTER', 90),
        'after_commit' => false,
    ],

    'redis' => [
        'driver' => 'redis',
        'connection' => env('REDIS_QUEUE_CONNECTION', 'default'),
        'queue' => env('REDIS_QUEUE', 'default'),
        'retry_after' => (int) env('REDIS_QUEUE_RETRY_AFTER', 90),
        'block_for' => null,
        'after_commit' => false,
    ],

    'sqs' => [
        'driver' => 'sqs',
        'key' => env('AWS_ACCESS_KEY_ID'),
        'secret' => env('AWS_SECRET_ACCESS_KEY'),
        'prefix' => env('SQS_PREFIX', 'https://sqs.us-east-1.amazonaws.com/your-account-id'),
        'queue' => env('SQS_QUEUE', 'default'),
        'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
        'after_commit' => false,
    ],

    'sync' => [
        'driver' => 'sync',  // Runs immediately (good for testing)
    ],
],
```

### Choosing a Driver

| Driver | Use Case |
|--------|----------|
| **sync** | Local development, testing |
| **database** | Simple apps, getting started |
| **redis** | Production apps, fast performance |
| **sqs** | AWS deployments, high scale |
| **beanstalkd** | Alternative to Redis |

## Database Queue Setup

Laravel 12 includes queue migrations by default:

```bash
# If you need to create them manually
php artisan make:queue-table
php artisan migrate
```

The jobs table structure:

```php
Schema::create('jobs', function (Blueprint $table) {
    $table->id();
    $table->string('queue')->index();
    $table->longText('payload');
    $table->unsignedTinyInteger('attempts');
    $table->unsignedInteger('reserved_at')->nullable();
    $table->unsignedInteger('available_at');
    $table->unsignedInteger('created_at');
});
```

## Configuring the Queue Connection

```env
# .env
QUEUE_CONNECTION=database

# For Redis
QUEUE_CONNECTION=redis
REDIS_HOST=127.0.0.1
REDIS_PASSWORD=null
REDIS_PORT=6379
```

## Running Queue Workers

```bash
# Start processing jobs
php artisan queue:work

# Process jobs from specific queue
php artisan queue:work --queue=high,default,low

# Process single job and exit
php artisan queue:work --once

# With configuration options
php artisan queue:work --tries=3 --timeout=60 --memory=128

# Listen mode (restarts on code changes)
php artisan queue:listen
```

## The Development Server

Laravel 12's `composer dev` runs everything together:

```bash
composer dev
# Runs: server, queue:listen, pail (logs), and vite concurrently
```

## Resources

- [Queues](https://laravel.com/docs/12.x/queues) — Official Laravel queue documentation

---

> 📘 *This lesson is part of the [Laravel Background Processing](https://stanza.dev/courses/laravel-queues-jobs) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*