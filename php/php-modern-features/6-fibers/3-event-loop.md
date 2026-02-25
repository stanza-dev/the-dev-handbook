---
source_course: "php-modern-features"
source_lesson: "php-modern-features-fibers-event-loop"
---

# Building an Event Loop with Fibers

## Introduction
Understanding how to build an event loop helps demystify async PHP libraries. An event loop is the central coordinator that manages fibers, timers, and I/O callbacks, running them cooperatively in a single thread.

## Key Concepts
- **Event Loop**: A loop that processes pending callbacks, checks timers, and resumes fibers until no work remains.
- **Deferred Execution**: Scheduling a callback to run on the next loop iteration rather than immediately.
- **Promise/Deferred Pattern**: A value that will be resolved in the future, with a fiber waiting for it.

## Real World Context
Every async PHP application runs inside an event loop. Revolt (the event loop behind Amp v3) and ReactPHP's event loop both implement the pattern shown here, with optimized I/O polling via `stream_select()`, `epoll`, or `libuv`. Building a simplified version reveals the core mechanics.

## Deep Dive

### Simple Event Loop

A minimal event loop with fibers, timers, and deferred callbacks:

```php
<?php
class EventLoop {
    private array $fibers = [];
    private array $timers = [];
    private array $pending = [];
    
    public function defer(callable $callback): void {
        $this->pending[] = $callback;
    }
    
    public function delay(float $seconds, callable $callback): void {
        $this->timers[] = [
            'time' => microtime(true) + $seconds,
            'callback' => $callback,
        ];
    }
    
    public function async(callable $callback): void {
        $this->fibers[] = new Fiber($callback);
    }
    
    public function run(): void {
        foreach ($this->fibers as $fiber) {
            if (!$fiber->isStarted()) {
                $fiber->start($this);
            }
        }
        
        while ($this->hasWork()) {
            foreach ($this->pending as $key => $callback) {
                unset($this->pending[$key]);
                $callback();
            }
            
            $now = microtime(true);
            foreach ($this->timers as $key => $timer) {
                if ($timer['time'] <= $now) {
                    unset($this->timers[$key]);
                    $timer['callback']();
                }
            }
            
            foreach ($this->fibers as $key => $fiber) {
                if ($fiber->isSuspended()) {
                    $fiber->resume();
                }
                if ($fiber->isTerminated()) {
                    unset($this->fibers[$key]);
                }
            }
            
            usleep(1000);  // Prevent CPU spinning
        }
    }
    
    private function hasWork(): bool {
        return !empty($this->fibers) || !empty($this->timers) || !empty($this->pending);
    }
}
```

The loop runs until all fibers have terminated, all timers have fired, and all pending callbacks have been executed.

### Async Sleep Implementation

Real async sleep cooperates with the event loop instead of blocking:

```php
<?php
function asyncSleep(EventLoop $loop, float $seconds): void {
    $fiber = Fiber::getCurrent();
    
    if ($fiber === null) {
        throw new RuntimeException('asyncSleep must be called from a Fiber');
    }
    
    $loop->delay($seconds, function() use ($fiber) {
        if ($fiber->isSuspended()) {
            $fiber->resume();
        }
    });
    
    Fiber::suspend();
}
```

The fiber suspends itself and registers a timer. When the timer fires, it resumes the fiber. Other fibers can run while this one waits.

### Using the Event Loop

Run concurrent tasks:

```php
<?php
$loop = new EventLoop();

$loop->async(function() use ($loop) {
    echo "Task 1: Start\n";
    asyncSleep($loop, 0.5);
    echo "Task 1: After 500ms\n";
});

$loop->async(function() use ($loop) {
    echo "Task 2: Start\n";
    asyncSleep($loop, 0.3);
    echo "Task 2: After 300ms\n";
});

$loop->run();
// Both tasks run concurrently!
// Task 2 completes first (shorter sleep)
```

Both tasks start immediately. Task 2's shorter sleep causes it to resume first, demonstrating cooperative concurrency.

### Promise-Like Pattern

A deferred value that fibers can await:

```php
<?php
class Deferred {
    private ?Fiber $waiting = null;
    private mixed $result = null;
    private bool $resolved = false;
    
    public function resolve(mixed $value): void {
        $this->result = $value;
        $this->resolved = true;
        if ($this->waiting?->isSuspended()) {
            $this->waiting->resume($value);
        }
    }
    
    public function await(): mixed {
        if ($this->resolved) {
            return $this->result;
        }
        $this->waiting = Fiber::getCurrent();
        return Fiber::suspend();
    }
}
```

A fiber calls `$deferred->await()` and suspends. Later, when `$deferred->resolve($value)` is called, the fiber resumes with the value.

## Common Pitfalls
1. **CPU spinning without usleep** — An event loop without a small sleep in each iteration consumes 100% CPU while waiting for timers. Always include a small delay.
2. **Forgetting to check isSuspended before resume** — A fiber might have terminated or might still be running. Always guard `resume()` with a state check.

## Best Practices
1. **Use production event loops** — Revolt and ReactPHP's event loops handle edge cases, I/O polling, and signal handling that this simplified version omits.
2. **Keep event loop callbacks small** — Long-running synchronous code in a callback blocks the entire loop. Break heavy work into fiber-suspended chunks.

## Summary
- An event loop manages fibers, timers, and deferred callbacks in a single thread.
- Async sleep suspends a fiber and registers a timer to resume it later.
- The Deferred/Promise pattern lets fibers await values resolved elsewhere.
- Real async libraries (Revolt, ReactPHP) build on these patterns with I/O polling.
- Always use production event loops for application code.

## Code Examples

**Deferred/Promise pattern where a fiber awaits a value that is resolved asynchronously**

```php
<?php
declare(strict_types=1);

class Deferred {
    private ?Fiber $waiting = null;
    private mixed $result = null;
    private bool $resolved = false;
    
    public function resolve(mixed $value): void {
        $this->result = $value;
        $this->resolved = true;
        if ($this->waiting?->isSuspended()) {
            $this->waiting->resume($value);
        }
    }
    
    public function await(): mixed {
        if ($this->resolved) return $this->result;
        $this->waiting = Fiber::getCurrent();
        return Fiber::suspend();
    }
}

// Usage
$deferred = new Deferred();

$fiber = new Fiber(function() use ($deferred) {
    echo "Waiting for result...\n";
    $result = $deferred->await();
    echo "Got: $result\n";
});

$fiber->start();          // Fiber suspends, waiting
$deferred->resolve(42);   // Fiber resumes with 42
// Output: Waiting for result...
//         Got: 42
?>
```


## Resources

- [Fibers](https://www.php.net/manual/en/language.fibers.php) — PHP Fibers documentation

---

> 📘 *This lesson is part of the [Modern PHP 8.x: Latest Language Features](https://stanza.dev/courses/php-modern-features) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*