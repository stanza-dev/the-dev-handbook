---
source_course: "php-async"
source_lesson: "php-async-fibers-introduction"
---

# Introduction to PHP Fibers

## Introduction
PHP 8.1 introduced Fibers, a revolutionary feature that enables cooperative multitasking without the complexity of callbacks or generators. Fibers are the foundation for modern async PHP libraries like ReactPHP and Amp, and understanding them unlocks a new level of PHP performance.
## Key Concepts
- **Fiber**: A lightweight execution context in PHP 8.1+ that can be paused (suspended) and resumed, enabling cooperative multitasking.
- **Fiber::suspend()**: Pauses the Fiber's execution and returns control to the caller, optionally passing a value out.
- **Fiber::resume()**: Continues a suspended Fiber's execution from where it left off, optionally passing a value in.
- **Stack Switching**: The mechanism Fibers use to save and restore the entire call stack, unlike Generators which only preserve the current function's state.
- **Fiber Lifecycle**: The states a Fiber transitions through: Created, Running, Suspended, and Terminated.
## Real World Context
Libraries like ReactPHP and Amp use Fibers internally to provide clean async/await-style APIs. Understanding how Fibers work helps you debug issues, write custom async utilities, and make informed architectural decisions.

## Deep Dive
## What is a Fiber?

A **Fiber** is a lightweight execution context that can be paused and resumed. Think of it as a function that can be suspended mid-execution and later continued from exactly where it left off.

```php
<?php
// Basic Fiber example
$fiber = new Fiber(function(): void {
    echo "Fiber started\n";
    
    $value = Fiber::suspend('paused');
    
    echo "Fiber resumed with: {$value}\n";
});

// Start the fiber
$result = $fiber->start();
echo "Fiber yielded: {$result}\n";

// Resume the fiber
$fiber->resume('hello');
```

Output:
```
Fiber started
Fiber yielded: paused
Fiber resumed with: hello
```

## The Fiber Lifecycle

```
┌─────────────┐
│   Created   │ ──start()──▶ ┌──────────┐
└─────────────┘               │ Running  │
                              └────┬─────┘
                                   │
              ┌────────────────────┼────────────────────┐
              │                    │                    │
              ▼                    ▼                    ▼
       ┌──────────┐         ┌──────────┐        ┌────────────┐
       │ Suspended│◀─resume─│  Running │        │ Terminated │
       └────┬─────┘         └──────────┘        └────────────┘
            │
            └──────resume()──────▶ (back to Running)
```

## Fiber States

| Method | Description |
|--------|-------------|
| `Fiber::getCurrent()` | Get the currently executing Fiber |
| `$fiber->start(...$args)` | Start execution, passing arguments |
| `$fiber->resume($value)` | Resume a suspended Fiber |
| `$fiber->throw($exception)` | Resume by throwing an exception |
| `$fiber->isStarted()` | Has the Fiber been started? |
| `$fiber->isSuspended()` | Is the Fiber currently suspended? |
| `$fiber->isRunning()` | Is the Fiber currently running? |
| `$fiber->isTerminated()` | Has the Fiber finished execution? |
| `$fiber->getReturn()` | Get the return value (after termination) |

## How Fibers Work Internally

Fibers use **stack switching** to save and restore execution state:

```php
<?php
$fiber = new Fiber(function(): string {
    $a = 1;                    // Local state preserved
    Fiber::suspend();
    
    $b = 2;                    // Continues with $a still = 1
    Fiber::suspend();
    
    return "a={$a}, b={$b}";   // Both variables intact
});

$fiber->start();
$fiber->resume();
$fiber->resume();

echo $fiber->getReturn();  // "a=1, b=2"
```

Unlike generators (which only preserve the current function's state), Fibers preserve the **entire call stack**:

```php
<?php
function innerFunction(): void {
    echo "Inner: before suspend\n";
    Fiber::suspend();  // Suspends through multiple call frames!
    echo "Inner: after suspend\n";
}

function outerFunction(): void {
    echo "Outer: before inner\n";
    innerFunction();  // Call stack preserved across suspend
    echo "Outer: after inner\n";
}

$fiber = new Fiber(outerFunction(...));
$fiber->start();   // Outputs: Outer: before inner, Inner: before suspend
$fiber->resume();  // Outputs: Inner: after suspend, Outer: after inner
```

## Passing Data Through Suspend/Resume

Data flows bidirectionally:

```php
<?php
$fiber = new Fiber(function(): int {
    // Suspend returns what resume() passes in
    $x = Fiber::suspend('waiting for x');
    $y = Fiber::suspend('waiting for y');
    
    return $x + $y;
});

$message1 = $fiber->start();    // Returns 'waiting for x'
echo $message1 . "\n";

$message2 = $fiber->resume(10); // Passes 10, returns 'waiting for y'
echo $message2 . "\n";

$fiber->resume(20);             // Passes 20, fiber completes

echo $fiber->getReturn();       // 30
```

## Exception Handling

Fibers handle exceptions naturally:

```php
<?php
$fiber = new Fiber(function(): void {
    try {
        Fiber::suspend();
    } catch (RuntimeException $e) {
        echo "Caught: " . $e->getMessage() . "\n";
    }
});

$fiber->start();

// Inject an exception into the suspended fiber
$fiber->throw(new RuntimeException('Something went wrong'));
// Outputs: Caught: Something went wrong
```

## Memory and Performance

Fibers are lightweight:

- Each Fiber has its own stack (default ~8KB)
- Creating thousands of Fibers is feasible
- Context switching is fast (microseconds)
- Memory is freed when Fiber is garbage collected

```php
<?php
// Creating many fibers is efficient
$fibers = [];
for ($i = 0; $i < 10000; $i++) {
    $fibers[] = new Fiber(function() use ($i): int {
        Fiber::suspend();
        return $i * 2;
    });
}

echo "Created 10,000 fibers\n";
echo "Memory: " . round(memory_get_usage() / 1024 / 1024, 2) . " MB\n";
```

## Common Pitfalls
1. **Using Fibers directly in application code** - Fibers are a low-level primitive. Use higher-level libraries like ReactPHP or Amp that abstract the complexity.
2. **Forgetting that Fibers are single-threaded** - A blocking operation inside a Fiber blocks the entire thread, defeating the purpose of concurrency.
3. **Not checking Fiber state before operations** - Calling resume() on a non-suspended Fiber or start() on an already-started Fiber throws FiberError.

## Best Practices
1. **Use libraries that abstract Fibers** - ReactPHP and Amp provide high-level APIs built on Fibers. Use those instead of raw Fiber operations.
2. **Keep Fiber callbacks short and focused** - Each Fiber should do one thing. Complex logic should be broken into multiple functions.
3. **Always handle the Fiber return value** - After a Fiber terminates, call getReturn() to retrieve its result or detect if it threw an exception.

## Summary
- Fibers are lightweight execution contexts that can be paused (suspended) and resumed.
- Unlike Generators, Fibers preserve the entire call stack across suspension points.
- Data flows bidirectionally through suspend() and resume() parameters.
- Fibers are memory-efficient (~8KB stack each) and support thousands of concurrent instances.
- In practice, use libraries like ReactPHP or Amp rather than raw Fibers.

## Resources

- [PHP Manual - Fibers](https://www.php.net/manual/en/language.fibers.php) — Official PHP documentation on Fibers

---

> 📘 *This lesson is part of the [Asynchronous PHP](https://stanza.dev/courses/php-async) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*