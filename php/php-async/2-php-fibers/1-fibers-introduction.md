---
source_course: "php-async"
source_lesson: "php-async-fibers-introduction"
---

# Introduction to PHP Fibers

PHP 8.1 introduced **Fibers**, a revolutionary feature that enables cooperative multitasking without the complexity of callbacks or generators. Fibers are the foundation for modern async PHP libraries.

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

## Resources

- [PHP Manual - Fibers](https://www.php.net/manual/en/language.fibers.php) — Official PHP documentation on Fibers

---

> 📘 *This lesson is part of the [Asynchronous PHP](https://stanza.dev/courses/php-async) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*