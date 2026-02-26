---
source_course: "php-async"
source_lesson: "php-async-concurrency-concepts"
---

# Concurrency, Parallelism, and Async

## Introduction
The terms concurrency, parallelism, and asynchronous are often confused, but understanding their differences is essential for writing efficient PHP applications. Choosing the wrong approach for your workload can mean wasted effort or, worse, introducing complexity without any performance benefit.
## Key Concepts
- **Concurrency**: Managing multiple tasks that can make progress over time by interleaving their execution, even on a single thread.
- **Parallelism**: Actually executing multiple tasks at the exact same time, requiring multiple CPU cores or threads.
- **Cooperative Multitasking**: A model where tasks voluntarily yield control (used by PHP Fibers), as opposed to preemptive multitasking where the scheduler forces switches.
- **Event Loop**: A programming construct that continuously checks for and dispatches events, forming the foundation of async programming.
- **Promises**: Objects representing a value that may not be available yet, providing a cleaner alternative to callbacks for async result handling.
## Real World Context
Modern web applications routinely call multiple APIs, databases, and services to render a single page. Understanding the difference between concurrency and parallelism helps you choose the right tool: Fibers for I/O-bound concurrency, or pcntl_fork for CPU-bound parallelism.

## Deep Dive
## Concurrency vs Parallelism

These are related but distinct concepts:

**Concurrency**: Managing multiple tasks that can make progress over time. Tasks may not run simultaneously—they take turns.

**Parallelism**: Actually executing multiple tasks at the exact same time, requiring multiple CPU cores.

## The Coffee Shop Analogy

Imagine a coffee shop:

**Sequential (No Concurrency)**:
- One barista handles everything
- Takes order → Makes drink → Serves → Takes next order
- Each customer waits for all previous customers

**Concurrent (Single Barista)**:
- One barista, but works smarter
- Takes order → Starts coffee brewing → Takes next order while waiting
- Multiple orders in progress, but one person switching between tasks

**Parallel (Multiple Baristas)**:
- Several baristas working simultaneously
- Each handles their own customer
- True simultaneous work

## PHP's Options

```php
<?php
// Sequential - what you normally write
function fetchSequentially(): array {
    $result1 = slowOperation1();  // Wait...
    $result2 = slowOperation2();  // Wait...
    $result3 = slowOperation3();  // Wait...
    
    return [$result1, $result2, $result3];
}

// Total time = sum of all operations
```

### Achieving Concurrency in PHP

PHP offers several mechanisms:

| Mechanism | Type | Use Case |
|-----------|------|----------|
| **Fibers** | Cooperative concurrency | I/O-bound operations |
| **pcntl_fork()** | Process-based parallelism | CPU-bound or isolated tasks |
| **ext-parallel** | True parallelism | Heavy computation |
| **ReactPHP/Amp** | Event-driven concurrency | Network applications |

## Understanding Event Loops

At the heart of asynchronous PHP is the **event loop**—a programming construct that waits for and dispatches events.

```php
<?php
// Conceptual event loop
while ($running) {
    // Check for completed I/O operations
    $events = checkForReadyEvents();
    
    // Handle each ready event
    foreach ($events as $event) {
        $event->callback();
    }
    
    // Brief pause to prevent CPU spinning
    usleep(1000);
}
```

The event loop enables **non-blocking I/O**:

1. Start an operation (e.g., HTTP request)
2. Instead of waiting, register a callback
3. Continue to other work
4. Event loop notifies you when operation completes
5. Your callback runs with the result

## Cooperative vs Preemptive Multitasking

**Cooperative Multitasking** (PHP Fibers):
- Tasks voluntarily yield control
- Task decides when to pause
- Simpler, no race conditions
- One task can block others if it doesn't yield

```php
<?php
// Cooperative - task explicitly yields
$fiber = new Fiber(function() {
    echo "Start\n";
    Fiber::suspend();  // Voluntarily pause
    echo "Resume\n";
});
```

**Preemptive Multitasking** (OS threads):
- System forces task switches
- Tasks can be interrupted anytime
- Complex, requires locks/synchronization
- Fair scheduling, no task can monopolize

## Asynchronous Programming Patterns

### Callbacks (Traditional)

```php
<?php
// Callback-based async (can lead to "callback hell")
$http->request('GET', '/users', function($response) {
    $http->request('GET', '/orders/' . $response->userId, function($orders) {
        $http->request('GET', '/details/' . $orders[0]->id, function($details) {
            // Deeply nested callbacks...
        });
    });
});
```

### Promises

```php
<?php
// Promise-based (more readable)
$http->requestAsync('GET', '/users')
    ->then(function($response) use ($http) {
        return $http->requestAsync('GET', '/orders/' . $response->userId);
    })
    ->then(function($orders) use ($http) {
        return $http->requestAsync('GET', '/details/' . $orders[0]->id);
    })
    ->then(function($details) {
        // Handle final result
    });
```

### Async/Await Style (with Fibers)

```php
<?php
// Async/await style (most readable)
async function fetchUserDetails($userId) {
    $user = await $http->get('/users/' . $userId);
    $orders = await $http->get('/orders/' . $user->id);
    $details = await $http->get('/details/' . $orders[0]->id);
    
    return $details;
}
```

## The PHP Execution Context

```
Traditional PHP Web Request:
┌─────────────────────────────────────────┐
│  Web Server (Apache/Nginx)              │
│  ┌─────────────────────────────────┐    │
│  │  PHP Process (per request)      │    │
│  │  - Single thread                │    │
│  │  - Synchronous by default       │    │
│  │  - Dies after response          │    │
│  └─────────────────────────────────┘    │
└─────────────────────────────────────────┘

Async PHP Server (ReactPHP/Swoole):
┌─────────────────────────────────────────┐
│  Long-running PHP Process               │
│  ┌─────────────────────────────────┐    │
│  │  Event Loop                     │    │
│  │  - Handles many connections     │    │
│  │  - Non-blocking I/O             │    │
│  │  - Runs indefinitely            │    │
│  └─────────────────────────────────┘    │
└─────────────────────────────────────────┘
```

## Common Pitfalls
1. **Confusing concurrency with parallelism** - Running Fibers does not make your code run on multiple CPU cores. Fibers provide concurrency (interleaved execution) on a single thread.
2. **Using async for CPU-bound tasks** - Async only helps when tasks spend time waiting. CPU-bound work needs true parallelism (pcntl_fork or ext-parallel).
3. **Overcomplicating simple flows** - If operations are sequential dependencies (B needs A's result), async adds complexity without benefit.

## Best Practices
1. **Choose the right concurrency model** - Use Fibers/event loops for I/O-bound work, pcntl_fork for CPU-bound work, and synchronous code for simple sequential tasks.
2. **Start with a high-level library** - Libraries like ReactPHP and Amp handle the complexity of Fibers and event loops so you can focus on business logic.
3. **Understand your bottleneck** - Determine whether your application is I/O-bound or CPU-bound before choosing a concurrency strategy.

## Summary
1. **Concurrency** ≠ **Parallelism**: Concurrency is about structure, parallelism is about execution
2. **Async** shines for I/O-bound work, not CPU-bound work
3. **Event loops** are the foundation of async programming
4. **Fibers** enable cooperative concurrency in PHP 8.1+
5. Modern PHP can handle high-concurrency scenarios with the right tools

## Resources

- [PHP RFC: Fibers](https://wiki.php.net/rfc/fibers) — The original RFC that introduced Fibers to PHP

---

> 📘 *This lesson is part of the [Asynchronous PHP](https://stanza.dev/courses/php-async) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*