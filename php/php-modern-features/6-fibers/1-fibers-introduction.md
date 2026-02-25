---
source_course: "php-modern-features"
source_lesson: "php-modern-features-fibers-introduction"
---

# Introduction to Fibers

## Introduction
Fibers, introduced in PHP 8.1, are lightweight threads that enable cooperative multitasking. They allow code to pause and resume execution at specific points, forming the foundation for asynchronous programming libraries in PHP.

## Key Concepts
- **Fiber**: A lightweight execution context that can suspend and resume, similar to coroutines in other languages.
- **Cooperative Multitasking**: Code explicitly yields control rather than being preempted by a scheduler.
- **Suspend/Resume**: A fiber pauses with `Fiber::suspend()` and continues where it left off when `$fiber->resume()` is called.

## Real World Context
Fibers power async PHP libraries like ReactPHP, Amp, and Revolt. They make it possible to write asynchronous code that looks synchronous — no callback hell, no promise chains. While you rarely use Fibers directly, understanding them helps you debug and extend async applications.

## Deep Dive

### Basic Fiber Example

Create and interact with a fiber:

```php
<?php
$fiber = new Fiber(function(): void {
    echo "1. Fiber started\n";
    Fiber::suspend('first suspension');
    echo "3. Fiber resumed\n";
    Fiber::suspend('second suspension');
    echo "5. Fiber finishing\n";
});

echo "Starting fiber...\n";
$result1 = $fiber->start();
echo "2. Received: $result1\n";

$result2 = $fiber->resume();
echo "4. Received: $result2\n";

$fiber->resume();
echo "6. Fiber completed\n";
```

The output follows the numbered sequence: the fiber and caller alternate control. Each `Fiber::suspend()` pauses the fiber and returns a value to the caller.

### Fiber Methods

The key methods for controlling fibers:

```php
<?php
$fiber = new Fiber(function() {
    $value = Fiber::suspend('paused');
    return "Got: $value";
});

// Start execution - runs until first suspend
$suspended = $fiber->start();
echo $suspended;  // 'paused'

// Resume with a value - the value becomes the return of suspend()
$result = $fiber->resume('hello');

// Check fiber state
$fiber->isStarted();    // Has start() been called?
$fiber->isRunning();    // Is currently executing?
$fiber->isSuspended();  // Is paused?
$fiber->isTerminated(); // Has finished?

// Get return value (only after termination)
$returnValue = $fiber->getReturn();
```

The `resume()` method can pass a value into the fiber, which becomes the return value of the `Fiber::suspend()` call inside the fiber.

### Static Methods

Fibers provide two static methods:

```php
<?php
// Inside a fiber callback
Fiber::suspend($value);    // Pause and return value to caller
$current = Fiber::getCurrent();  // Get current Fiber instance (or null)
```

`Fiber::getCurrent()` returns `null` when called from the main execution context (outside any fiber).

### Important Concepts

Four key facts about fibers:

1. **Only one fiber runs at a time** — there is no true parallelism, just cooperative switching.
2. **Explicit suspension required** — a fiber runs until it calls `Fiber::suspend()` or returns.
3. **Foundation for async libraries** — fibers are typically wrapped by higher-level async APIs.
4. **Exceptions can cross the boundary** — both the fiber and the caller can throw exceptions to each other.

## Common Pitfalls
1. **Expecting parallel execution** — Fibers are cooperative, not parallel. Only one runs at a time. If a fiber blocks on I/O synchronously, everything blocks.
2. **Calling `resume()` on a terminated fiber** — Throws a `FiberError`. Always check `isTerminated()` before resuming.

## Best Practices
1. **Use async libraries, not raw fibers** — Libraries like Amp and ReactPHP provide proper event loops, timers, and I/O integration on top of fibers. Use those unless you are building a framework.
2. **Think of fibers as implementation details** — Application code should use `async`/`await` patterns provided by libraries, not `Fiber::suspend()` directly.

## Summary
- Fibers are lightweight cooperative threads that can suspend and resume.
- Use `Fiber::suspend()` to pause and `$fiber->resume()` to continue.
- Values can be passed in both directions: suspend returns to caller, resume passes into fiber.
- Fibers power async PHP libraries but are rarely used directly.
- Only one fiber runs at a time — no true parallelism.

## Code Examples

**Fibonacci sequence generator using a Fiber that yields values one at a time via suspend()**

```php
<?php
function fibonacciGenerator(int $count): Fiber {
    return new Fiber(function() use ($count): void {
        $a = 0;
        $b = 1;
        
        for ($i = 0; $i < $count; $i++) {
            Fiber::suspend($a);
            [$a, $b] = [$b, $a + $b];
        }
    });
}

$fib = fibonacciGenerator(10);
$numbers = [];

$numbers[] = $fib->start();
while (!$fib->isTerminated()) {
    $val = $fib->resume();
    if (!$fib->isTerminated()) {
        $numbers[] = $val;
    }
}

print_r($numbers);
// [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]
?>
```


## Resources

- [Fibers Overview](https://www.php.net/manual/en/language.fibers.php) — Official PHP Fibers documentation

---

> 📘 *This lesson is part of the [Modern PHP 8.x: Latest Language Features](https://stanza.dev/courses/php-modern-features) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*