---
source_course: "php-async"
source_lesson: "php-async-async-use-cases"
---

# Real-World Async Use Cases

Understanding when to use asynchronous programming is just as important as knowing how. Let's explore practical scenarios where async PHP provides significant benefits.

## Use Case 1: API Aggregation

Modern applications often need to combine data from multiple sources:

```php
<?php
// BEFORE: Sequential API calls
class DashboardService {
    public function getDashboardData(int $userId): array {
        // Each call blocks until complete
        $user = $this->userApi->getUser($userId);           // 200ms
        $orders = $this->orderApi->getOrders($userId);      // 300ms
        $notifications = $this->notificationApi->get($userId); // 150ms
        $recommendations = $this->recApi->get($userId);     // 400ms
        
        return compact('user', 'orders', 'notifications', 'recommendations');
        // Total: ~1050ms
    }
}

// AFTER: Concurrent API calls
class AsyncDashboardService {
    public function getDashboardData(int $userId): array {
        // All requests start immediately
        $promises = [
            'user' => $this->userApi->getUserAsync($userId),
            'orders' => $this->orderApi->getOrdersAsync($userId),
            'notifications' => $this->notificationApi->getAsync($userId),
            'recommendations' => $this->recApi->getAsync($userId),
        ];
        
        // Wait for all to complete
        return Promise\all($promises)->wait();
        // Total: ~400ms (slowest request)
    }
}
```

Result: **60% faster** dashboard load time.

## Use Case 2: Webhook Processing

When an event occurs, you might need to notify multiple external services:

```php
<?php
// Async webhook dispatcher
class WebhookDispatcher {
    public function dispatch(Event $event, array $endpoints): void {
        $payload = json_encode($event->toArray());
        
        // Fire all webhooks concurrently
        $promises = [];
        foreach ($endpoints as $endpoint) {
            $promises[] = $this->httpClient->postAsync(
                $endpoint->url,
                ['body' => $payload, 'timeout' => 5]
            );
        }
        
        // Don't wait for responses (fire-and-forget)
        // Or wait with timeout for acknowledgment
        Promise\settle($promises)->wait();
    }
}
```

## Use Case 3: Real-Time Chat Application

```php
<?php
// WebSocket chat server with ReactPHP
require 'vendor/autoload.php';

use Ratchet\MessageComponentInterface;
use Ratchet\ConnectionInterface;

class ChatServer implements MessageComponentInterface {
    protected \SplObjectStorage $clients;
    
    public function __construct() {
        $this->clients = new \SplObjectStorage();
    }
    
    public function onOpen(ConnectionInterface $conn): void {
        $this->clients->attach($conn);
        echo "New connection: {$conn->resourceId}\n";
    }
    
    public function onMessage(ConnectionInterface $from, $msg): void {
        // Broadcast to all connected clients
        foreach ($this->clients as $client) {
            if ($from !== $client) {
                $client->send($msg);
            }
        }
    }
    
    public function onClose(ConnectionInterface $conn): void {
        $this->clients->detach($conn);
    }
    
    public function onError(ConnectionInterface $conn, \Exception $e): void {
        $conn->close();
    }
}
```

## Use Case 4: Background Job Processing

```php
<?php
// Async queue worker
class AsyncWorker {
    public function run(): void {
        $loop = React\EventLoop\Loop::get();
        
        // Check for jobs every 100ms
        $loop->addPeriodicTimer(0.1, function() {
            $job = $this->queue->pop();
            
            if ($job) {
                $this->processAsync($job);
            }
        });
        
        // Process multiple jobs concurrently
        $loop->run();
    }
    
    private function processAsync(Job $job): void {
        // Start job without blocking
        $this->executor->executeAsync($job)
            ->then(
                fn($result) => $this->onSuccess($job, $result),
                fn($error) => $this->onFailure($job, $error)
            );
    }
}
```

## Use Case 5: Web Scraping

```php
<?php
// Concurrent web scraper
class WebScraper {
    public function scrapeUrls(array $urls): array {
        $results = [];
        $promises = [];
        
        foreach ($urls as $url) {
            $promises[$url] = $this->httpClient->getAsync($url)
                ->then(function ($response) use ($url) {
                    return [
                        'url' => $url,
                        'status' => $response->getStatusCode(),
                        'content' => $this->parseContent($response->getBody())
                    ];
                })
                ->otherwise(function ($error) use ($url) {
                    return [
                        'url' => $url,
                        'error' => $error->getMessage()
                    ];
                });
        }
        
        return Promise\settle($promises)->wait();
    }
}

// Scrape 100 pages concurrently (with connection pooling)
$scraper = new WebScraper();
$results = $scraper->scrapeUrls($hundredUrls);
// Takes seconds instead of minutes
```

## Use Case 6: Database Operations

```php
<?php
// Parallel database queries
class ReportGenerator {
    public function generateReport(DateRange $range): Report {
        // These queries can run simultaneously
        $promises = [
            'sales' => $this->db->queryAsync(
                'SELECT SUM(amount) FROM sales WHERE date BETWEEN ? AND ?',
                [$range->start, $range->end]
            ),
            'orders' => $this->db->queryAsync(
                'SELECT COUNT(*) FROM orders WHERE created_at BETWEEN ? AND ?',
                [$range->start, $range->end]
            ),
            'customers' => $this->db->queryAsync(
                'SELECT COUNT(DISTINCT customer_id) FROM orders WHERE created_at BETWEEN ? AND ?',
                [$range->start, $range->end]
            ),
        ];
        
        $data = Promise\all($promises)->wait();
        
        return new Report($data);
    }
}
```

## When NOT to Use Async

❌ **Simple CRUD operations**: Overhead isn't worth it
❌ **Sequential dependencies**: When each step needs the previous result
❌ **CPU-intensive tasks**: Async doesn't help compute-bound work
❌ **Short scripts**: Setup overhead exceeds benefits
❌ **Traditional web requests**: Request/response already fast enough

## Decision Framework

```
Should I use async?
        │
        ▼
┌─────────────────────────────┐
│ Multiple independent I/O    │──No──▶ Use synchronous
│ operations?                 │
└─────────────────────────────┘
        │ Yes
        ▼
┌─────────────────────────────┐
│ Operations take >100ms each │──No──▶ Probably not worth it
│ and there are 3+ of them?   │
└─────────────────────────────┘
        │ Yes
        ▼
┌─────────────────────────────┐
│ Performance is critical     │──No──▶ Consider readability
│ for this feature?           │
└─────────────────────────────┘
        │ Yes
        ▼
    Use async! ✅
```

## Resources

- [ReactPHP - Event-driven PHP](https://reactphp.org/) — Official ReactPHP documentation and examples

---

> 📘 *This lesson is part of the [Asynchronous PHP](https://stanza.dev/courses/php-async) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*