---
source_course: "php-modern-features"
source_lesson: "php-modern-features-fibers-practical"
---

# Practical Fiber Patterns

## Introduction
While you rarely use Fibers directly, understanding practical patterns helps when working with async libraries and debugging fiber-based applications. This lesson covers schedulers, exception handling, and how async libraries use fibers under the hood.

## Key Concepts
- **Scheduler**: A loop that manages multiple fibers, starting and resuming them in sequence.
- **Exception Injection**: Throwing an exception into a suspended fiber with `$fiber->throw()`.
- **Async Sleep**: A pattern where fibers cooperate with a timer system instead of blocking the entire process.

## Real World Context
Every async PHP library (ReactPHP, Amp, Revolt) uses these patterns internally. When you write `$response = await httpGet($url)`, the library suspends your fiber, registers an I/O callback, and resumes the fiber when data arrives. Understanding this mechanism helps you debug hangs, deadlocks, and unexpected behavior.

## Deep Dive

### Simple Scheduler

A scheduler runs multiple fibers cooperatively:

```php
<?php
class SimpleScheduler {
    private array $fibers = [];
    
    public function add(callable $task): void {
        $this->fibers[] = new Fiber($task);
    }
    
    public function run(): void {
        foreach ($this->fibers as $fiber) {
            $fiber->start();
        }
        
        while ($this->hasRunning()) {
            foreach ($this->fibers as $fiber) {
                if ($fiber->isSuspended()) {
                    $fiber->resume();
                }
            }
        }
    }
    
    private function hasRunning(): bool {
        foreach ($this->fibers as $fiber) {
            if (!$fiber->isTerminated()) {
                return true;
            }
        }
        return false;
    }
}
```

The scheduler starts all fibers, then loops through them, resuming suspended ones until all complete.

### Exception Handling

You can throw exceptions into a suspended fiber:

```php
<?php
$fiber = new Fiber(function() {
    try {
        Fiber::suspend('waiting');
    } catch (Exception $e) {
        echo "Caught: " . $e->getMessage();
        return 'handled';
    }
});

$fiber->start();

// Throw exception into the fiber
$fiber->throw(new Exception('Something went wrong'));
$result = $fiber->getReturn();
echo $result;  // 'handled'
```

The `throw()` method resumes the fiber, but the `Fiber::suspend()` call throws the given exception instead of returning normally. The fiber can catch and handle it.

### How Async Libraries Use Fibers

Conceptually, async I/O works like this:

```php
<?php
// What you write (high-level async API):
function fetchData(): string {
    $response = await httpGet('https://api.example.com');
    return $response->body;
}

// What the library does (simplified):
function httpGet(string $url): Response {
    $fiber = Fiber::getCurrent();
    
    // Register async I/O callback
    registerIoCallback($url, function($response) use ($fiber) {
        $fiber->resume($response);
    });
    
    // Suspend until callback resumes us
    return Fiber::suspend();
}
```

The library suspends the fiber, registers an I/O callback with the event loop, and resumes the fiber when data arrives. The user code reads as synchronous.

### When to Use Fibers Directly

Fibers are appropriate for:

1. **Building async frameworks** — If you are creating an event loop or async runtime.
2. **Cooperative multitasking** — Running multiple computations that voluntarily yield.
3. **Coroutine-like patterns** — Generators on steroids with bidirectional communication.

For application code, use libraries built on fibers (Amp, ReactPHP) instead.

## Common Pitfalls
1. **Resuming a terminated fiber** — Calling `resume()` or `throw()` on a terminated fiber throws a `FiberError`. Always check `isTerminated()` first.
2. **Suspending outside a fiber** — Calling `Fiber::suspend()` from the main execution context throws a `FiberError`. Only call it from within a fiber callback.

## Best Practices
1. **Check fiber state before operations** — Always check `isSuspended()` before `resume()` and `isTerminated()` before accessing `getReturn()`.
2. **Use structured concurrency** — Track all spawned fibers and ensure they complete or are cancelled before the parent scope exits.

## Summary
- A scheduler manages multiple fibers by starting and resuming them in a loop.
- `$fiber->throw()` injects exceptions into suspended fibers.
- Async libraries use fibers to make I/O non-blocking while keeping user code synchronous.
- Use async libraries (Amp, ReactPHP) for application code, not raw fibers.
- Always check fiber state before resume/throw operations.

## Code Examples

**Concurrent task runner that interleaves multiple fibers and collects their results**

```php
<?php
class TaskRunner {
    private array $tasks = [];
    private array $results = [];
    
    public function addTask(string $name, callable $task): self {
        $this->tasks[$name] = new Fiber($task);
        return $this;
    }
    
    public function run(): array {
        foreach ($this->tasks as $name => $fiber) {
            $fiber->start();
        }
        
        $pending = count($this->tasks);
        while ($pending > 0) {
            foreach ($this->tasks as $name => $fiber) {
                if ($fiber->isSuspended()) {
                    $fiber->resume();
                }
                if ($fiber->isTerminated() && !isset($this->results[$name])) {
                    $this->results[$name] = $fiber->getReturn();
                    $pending--;
                }
            }
        }
        return $this->results;
    }
}

$runner = new TaskRunner();
$runner->addTask('task1', function() {
    for ($i = 0; $i < 3; $i++) { Fiber::suspend(); }
    return 'Task 1 done';
});
$runner->addTask('task2', function() {
    for ($i = 0; $i < 2; $i++) { Fiber::suspend(); }
    return 'Task 2 done';
});

$results = $runner->run();
print_r($results);
// ['task1' => 'Task 1 done', 'task2' => 'Task 2 done']
?>
```


## Resources

- [Fiber Class](https://www.php.net/manual/en/class.fiber.php) — Complete Fiber class reference

---

> 📘 *This lesson is part of the [Modern PHP 8.x: Latest Language Features](https://stanza.dev/courses/php-modern-features) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*