---
source_course: "php-async"
source_lesson: "php-async-fiber-practical-patterns"
---

# Practical Fiber Patterns

## Introduction
While you typically will not use Fibers directly in application code (libraries like ReactPHP and Amp abstract them), understanding common Fiber patterns helps you use async libraries effectively and debug issues when they arise.
## Key Concepts
- **Scheduler**: A component that manages multiple Fibers, deciding which to start, resume, or terminate based on their state.
- **Async Sleep**: A non-blocking alternative to sleep() that suspends a Fiber and uses timers to schedule its resumption, allowing other Fibers to run.
- **Async/Await Simulation**: A pattern combining Fibers and Promises to write async code that looks and reads like synchronous code.
- **Generator-to-Fiber Migration**: The process of converting older generator-based async code to the cleaner Fiber-based approach introduced in PHP 8.1.
## Real World Context
Libraries like ReactPHP and Amp use Fibers internally to provide clean async/await-style APIs. Understanding how Fibers work helps you debug issues, write custom async utilities, and make informed architectural decisions.

## Deep Dive
## Pattern 1: Simple Scheduler

A scheduler manages multiple Fibers, deciding which to run:

```php
<?php
class SimpleScheduler {
    /** @var SplQueue<Fiber> */
    private SplQueue $queue;
    
    public function __construct() {
        $this->queue = new SplQueue();
    }
    
    public function add(callable $task): void {
        $this->queue->enqueue(new Fiber($task));
    }
    
    public function run(): void {
        while (!$this->queue->isEmpty()) {
            $fiber = $this->queue->dequeue();
            
            try {
                if (!$fiber->isStarted()) {
                    $fiber->start();
                } elseif ($fiber->isSuspended()) {
                    $fiber->resume();
                }
                
                // If still running (suspended), requeue
                if (!$fiber->isTerminated()) {
                    $this->queue->enqueue($fiber);
                }
            } catch (Throwable $e) {
                echo "Task failed: " . $e->getMessage() . "\n";
            }
        }
    }
}

// Usage
$scheduler = new SimpleScheduler();

$scheduler->add(function() {
    echo "Task 1: Step 1\n";
    Fiber::suspend();
    echo "Task 1: Step 2\n";
    Fiber::suspend();
    echo "Task 1: Done\n";
});

$scheduler->add(function() {
    echo "Task 2: Step 1\n";
    Fiber::suspend();
    echo "Task 2: Done\n";
});

$scheduler->run();
```

Output:
```
Task 1: Step 1
Task 2: Step 1
Task 1: Step 2
Task 2: Done
Task 1: Done
```

## Pattern 2: Async Sleep

```php
<?php
class AsyncRuntime {
    private array $timers = [];
    private SplQueue $ready;
    
    public function __construct() {
        $this->ready = new SplQueue();
    }
    
    public function delay(float $seconds): void {
        $fiber = Fiber::getCurrent();
        $resumeAt = microtime(true) + $seconds;
        
        $this->timers[] = [
            'fiber' => $fiber,
            'time' => $resumeAt
        ];
        
        // Sort by time
        usort($this->timers, fn($a, $b) => $a['time'] <=> $b['time']);
        
        Fiber::suspend();
    }
    
    public function spawn(callable $task): void {
        $this->ready->enqueue(new Fiber($task));
    }
    
    public function run(): void {
        while (!$this->ready->isEmpty() || !empty($this->timers)) {
            // Process ready fibers
            while (!$this->ready->isEmpty()) {
                $fiber = $this->ready->dequeue();
                
                if (!$fiber->isStarted()) {
                    $fiber->start();
                } elseif ($fiber->isSuspended()) {
                    $fiber->resume();
                }
            }
            
            // Check timers
            $now = microtime(true);
            foreach ($this->timers as $key => $timer) {
                if ($timer['time'] <= $now) {
                    $this->ready->enqueue($timer['fiber']);
                    unset($this->timers[$key]);
                }
            }
            $this->timers = array_values($this->timers);
            
            // Small sleep to prevent CPU spinning
            if ($this->ready->isEmpty() && !empty($this->timers)) {
                usleep(1000);
            }
        }
    }
}

// Usage
$runtime = new AsyncRuntime();

$runtime->spawn(function() use ($runtime) {
    echo "[" . date('H:i:s') . "] Task 1: Starting\n";
    $runtime->delay(2);
    echo "[" . date('H:i:s') . "] Task 1: After 2 seconds\n";
});

$runtime->spawn(function() use ($runtime) {
    echo "[" . date('H:i:s') . "] Task 2: Starting\n";
    $runtime->delay(1);
    echo "[" . date('H:i:s') . "] Task 2: After 1 second\n";
});

$runtime->run();
```

## Pattern 3: Async/Await Simulation

```php
<?php
class Promise {
    private mixed $value = null;
    private ?Throwable $error = null;
    private bool $resolved = false;
    private array $callbacks = [];
    
    public function resolve(mixed $value): void {
        $this->value = $value;
        $this->resolved = true;
        foreach ($this->callbacks as $callback) {
            $callback($value);
        }
    }
    
    public function reject(Throwable $error): void {
        $this->error = $error;
        $this->resolved = true;
    }
    
    public function then(callable $callback): self {
        if ($this->resolved && $this->error === null) {
            $callback($this->value);
        } else {
            $this->callbacks[] = $callback;
        }
        return $this;
    }
    
    public function isResolved(): bool {
        return $this->resolved;
    }
    
    public function getValue(): mixed {
        if ($this->error) {
            throw $this->error;
        }
        return $this->value;
    }
}

function await(Promise $promise): mixed {
    $fiber = Fiber::getCurrent();
    
    if ($fiber === null) {
        throw new LogicException('await must be called within a Fiber');
    }
    
    if ($promise->isResolved()) {
        return $promise->getValue();
    }
    
    $promise->then(function($value) use ($fiber) {
        if ($fiber->isSuspended()) {
            $fiber->resume($value);
        }
    });
    
    return Fiber::suspend();
}

// Usage example
function fetchUserAsync(int $id): Promise {
    $promise = new Promise();
    
    // Simulate async operation
    // In real code, this would use non-blocking I/O
    $promise->resolve(['id' => $id, 'name' => "User {$id}"]);
    
    return $promise;
}

$fiber = new Fiber(function() {
    echo "Fetching user...\n";
    $user = await(fetchUserAsync(42));
    echo "Got user: " . $user['name'] . "\n";
});

$fiber->start();
```

## Pattern 4: Generator to Fiber Migration

If you have generator-based async code, here's how to migrate:

```php
<?php
// OLD: Generator-based
function oldAsyncTask(): Generator {
    $result1 = yield fetchData();
    $result2 = yield processData($result1);
    return $result2;
}

// NEW: Fiber-based
function newAsyncTask(): mixed {
    $result1 = await(fetchDataAsync());
    $result2 = await(processDataAsync($result1));
    return $result2;
}
```

The Fiber version reads more naturally—like synchronous code.

## Common Pitfalls
1. **Using Fibers directly in application code** - Fibers are a low-level primitive. Use higher-level libraries like ReactPHP or Amp that abstract the complexity.
2. **Forgetting that Fibers are single-threaded** - A blocking operation inside a Fiber blocks the entire thread, defeating the purpose of concurrency.
3. **Not checking Fiber state before operations** - Calling resume() on a non-suspended Fiber or start() on an already-started Fiber throws FiberError.

## Best Practices
1. **Don't use Fibers directly in application code**—use libraries like Amp or ReactPHP
2. **Always check Fiber state** before calling start/resume
3. **Handle exceptions** both inside and outside Fibers
4. **Avoid blocking operations** inside Fibers—they block the entire thread
5. **Use Fiber::getCurrent()** to check if you're inside a Fiber

## Summary
- Fibers are lightweight execution contexts that can be paused (suspended) and resumed.
- Unlike Generators, Fibers preserve the entire call stack across suspension points.
- Data flows bidirectionally through suspend() and resume() parameters.
- Fibers are memory-efficient (~8KB stack each) and support thousands of concurrent instances.
- In practice, use libraries like ReactPHP or Amp rather than raw Fibers.

## Resources

- [Amp - Async Framework](https://amphp.org/) — Amp framework documentation built on Fibers

---

> 📘 *This lesson is part of the [Asynchronous PHP](https://stanza.dev/courses/php-async) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*