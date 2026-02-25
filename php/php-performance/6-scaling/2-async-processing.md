---
source_course: "php-performance"
source_lesson: "php-performance-async-processing"
---

# Asynchronous Processing

## Introduction
Move slow operations out of the request cycle.

## Key Concepts
- **Message Queues**: Decouple time-consuming work from the request cycle (e.g., RabbitMQ, Redis queues, Amazon SQS).
- **Background Workers**: Long-running PHP processes (Symfony Messenger, Laravel Queue) that consume and process queued jobs.
- **Event-Driven Architecture**: Publishing events (e.g., `UserRegistered`) that trigger asynchronous handlers for emails, analytics, etc.
- **Fire and Forget**: Queueing work and returning an immediate response to the user, processing the work asynchronously.

## Real World Context
Every major PHP application uses async processing. When a user places an order, you immediately return a confirmation while background workers handle payment processing, inventory updates, email notifications, and analytics tracking. This keeps response times under 200ms regardless of how much work the order triggers.

## Deep Dive
### Intro

Move slow operations out of the request cycle.

### Message queues

```php
<?php
// Instead of processing immediately:
// POST /orders -> process payment -> send email -> respond

// Queue work for later:
// POST /orders -> queue job -> respond immediately

class QueueClient
{
    private Redis $redis;
    
    public function push(string $queue, array $job): void
    {
        $this->redis->rPush($queue, json_encode([
            'id' => uniqid(),
            'payload' => $job,
            'created_at' => time(),
        ]));
    }
    
    public function pop(string $queue, int $timeout = 0): ?array
    {
        $result = $this->redis->blPop($queue, $timeout);
        
        if ($result) {
            return json_decode($result[1], true);
        }
        
        return null;
    }
}

// In request handler
$queue->push('emails', [
    'type' => 'order_confirmation',
    'user_id' => $user->id,
    'order_id' => $order->id,
]);

return new JsonResponse(['order_id' => $order->id]);
```

### Worker process

```php
<?php
// worker.php - runs continuously
class Worker
{
    public function __construct(
        private QueueClient $queue,
        private array $handlers
    ) {}
    
    public function run(string $queueName): never
    {
        echo "Worker started on queue: $queueName\n";
        
        while (true) {
            $job = $this->queue->pop($queueName, timeout: 30);
            
            if ($job === null) {
                continue;  // Timeout, check again
            }
            
            try {
                $this->process($job);
                echo "Processed job: {$job['id']}\n";
            } catch (Throwable $e) {
                echo "Failed job: {$job['id']} - {$e->getMessage()}\n";
                $this->handleFailure($job, $e);
            }
        }
    }
    
    private function process(array $job): void
    {
        $type = $job['payload']['type'];
        
        if (!isset($this->handlers[$type])) {
            throw new RuntimeException("Unknown job type: $type");
        }
        
        $this->handlers[$type]->handle($job['payload']);
    }
    
    private function handleFailure(array $job, Throwable $e): void
    {
        // Move to failed queue or retry
        $this->queue->push('failed', [
            ...$job,
            'error' => $e->getMessage(),
            'failed_at' => time(),
        ]);
    }
}

// Run workers
$worker = new Worker($queue, [
    'order_confirmation' => new SendOrderConfirmation(),
    'process_payment' => new ProcessPayment(),
    'generate_report' => new GenerateReport(),
]);

$worker->run('default');
```

### Supervisor configuration

```ini
; /etc/supervisor/conf.d/worker.conf
[program:php-worker]
command=php /var/www/app/worker.php
process_name=%(program_name)s_%(process_num)02d
numprocs=4  ; Run 4 worker processes
autostart=true
autorestart=true
user=www-data
stdout_logfile=/var/log/worker.log
```

## Common Pitfalls
1. **Processing everything synchronously** — Sending emails, generating PDFs, and calling external APIs during the HTTP request blocks the user. Queue everything that doesn't need an immediate response.
2. **Not handling worker failures** — Queue workers can crash or be killed by OOM. Implement retry logic, dead letter queues, and idempotent job handlers to handle failures gracefully.

## Best Practices
1. **Queue anything that takes more than 100ms** — Email sending, image processing, report generation, and third-party API calls should all be processed asynchronously.
2. **Make queue jobs idempotent** — Jobs should produce the same result if executed twice. This allows safe retry on failure without creating duplicate side effects.

## Summary
- Message queues decouple slow operations from the HTTP request cycle, keeping response times fast.
- Use background workers for email, PDF generation, API calls, and any task taking more than 100ms.
- Design idempotent jobs with retry logic and dead letter queues for reliability.

## Code Examples

**Complete job queue with retries and delayed jobs**

```php
<?php
declare(strict_types=1);

// Complete job queue system
interface JobHandler {
    public function handle(array $payload): void;
}

class SendEmailHandler implements JobHandler {
    public function handle(array $payload): void {
        $email = $payload['email'];
        $subject = $payload['subject'];
        $body = $payload['body'];
        
        // Send email...
        mail($email, $subject, $body);
    }
}

class JobQueue {
    public function __construct(
        private Redis $redis,
        private string $prefix = 'queue:'
    ) {}
    
    public function dispatch(string $queue, string $handler, array $payload, int $delay = 0): string {
        $jobId = bin2hex(random_bytes(16));
        
        $job = [
            'id' => $jobId,
            'handler' => $handler,
            'payload' => $payload,
            'attempts' => 0,
            'created_at' => time(),
        ];
        
        if ($delay > 0) {
            // Delayed job
            $this->redis->zAdd(
                $this->prefix . 'delayed',
                time() + $delay,
                json_encode($job)
            );
        } else {
            $this->redis->rPush($this->prefix . $queue, json_encode($job));
        }
        
        return $jobId;
    }
    
    public function migrateDelayed(): int {
        $now = time();
        $jobs = $this->redis->zRangeByScore(
            $this->prefix . 'delayed',
            '-inf',
            (string) $now
        );
        
        foreach ($jobs as $jobJson) {
            $job = json_decode($jobJson, true);
            $this->redis->rPush($this->prefix . 'default', $jobJson);
            $this->redis->zRem($this->prefix . 'delayed', $jobJson);
        }
        
        return count($jobs);
    }
}

class JobWorker {
    private bool $running = true;
    
    public function __construct(
        private Redis $redis,
        private array $handlers,
        private int $maxAttempts = 3
    ) {
        pcntl_signal(SIGTERM, fn() => $this->running = false);
        pcntl_signal(SIGINT, fn() => $this->running = false);
    }
    
    public function run(string $queue = 'default'): void {
        $prefix = 'queue:';
        
        while ($this->running) {
            pcntl_signal_dispatch();
            
            // Migrate delayed jobs
            (new JobQueue($this->redis))->migrateDelayed();
            
            // Fetch job
            $result = $this->redis->blPop($prefix . $queue, 5);
            
            if (!$result) continue;
            
            $job = json_decode($result[1], true);
            $this->processJob($job, $queue);
        }
    }
    
    private function processJob(array $job, string $queue): void {
        $job['attempts']++;
        
        try {
            $handler = $this->handlers[$job['handler']] ?? null;
            
            if (!$handler) {
                throw new RuntimeException("Unknown handler: {$job['handler']}");
            }
            
            $handler->handle($job['payload']);
            
            echo "[OK] Job {$job['id']} processed\n";
            
        } catch (Throwable $e) {
            echo "[ERROR] Job {$job['id']}: {$e->getMessage()}\n";
            
            if ($job['attempts'] < $this->maxAttempts) {
                // Retry with exponential backoff
                $delay = pow(2, $job['attempts']) * 60;
                (new JobQueue($this->redis))->dispatch(
                    $queue,
                    $job['handler'],
                    $job['payload'],
                    $delay
                );
            } else {
                // Move to failed queue
                $job['error'] = $e->getMessage();
                $job['failed_at'] = time();
                $this->redis->rPush('queue:failed', json_encode($job));
            }
        }
    }
}

// Usage
$redis = new Redis();
$redis->connect('127.0.0.1');

// Dispatch jobs
$queue = new JobQueue($redis);
$queue->dispatch('default', SendEmailHandler::class, [
    'email' => 'user@example.com',
    'subject' => 'Welcome!',
    'body' => 'Thanks for signing up.',
]);

// Run worker
$worker = new JobWorker($redis, [
    SendEmailHandler::class => new SendEmailHandler(),
]);
$worker->run();
?>
```


## Resources

- [PCNTL Process Control](https://www.php.net/manual/en/book.pcntl.php) — PHP process control functions for workers

---

> 📘 *This lesson is part of the [PHP Performance Optimization](https://stanza.dev/courses/php-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*