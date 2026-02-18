---
source_course: "php-modern-features"
source_lesson: "php-modern-features-fibers-introduction"
---

# Introduction to Fibers (PHP 8.1+)

Fibers are lightweight threads that enable cooperative multitasking. They allow code to pause and resume execution, enabling asynchronous programming patterns.

## What Are Fibers?

- **Lightweight**: Minimal overhead compared to OS threads
- **Cooperative**: Code explicitly yields control
- **Synchronous-looking async**: Write async code that looks synchronous

## Basic Fiber Example

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

// Output:
// Starting fiber...
// 1. Fiber started
// 2. Received: first suspension
// 3. Fiber resumed
// 4. Received: second suspension
// 5. Fiber finishing
// 6. Fiber completed
```

## Fiber Methods

```php
<?php
$fiber = new Fiber(function() {
    $value = Fiber::suspend('paused');
    return "Got: $value";
});

// Start execution
$suspended = $fiber->start();
echo $suspended;  // 'paused'

// Resume with a value
$result = $fiber->resume('hello');
echo $result;  // 'Got: hello'

// Check fiber state
$fiber->isStarted();    // Has start() been called?
$fiber->isRunning();    // Is currently executing?
$fiber->isSuspended();  // Is paused?
$fiber->isTerminated(); // Has finished?

// Get return value (after termination)
$returnValue = $fiber->getReturn();
```

## Static Methods

```php
<?php
// Inside a fiber callback
Fiber::suspend($value);    // Pause and return value to caller
Fiber::getCurrent();       // Get current Fiber instance (or null)

// Example
$fiber = new Fiber(function() {
    $current = Fiber::getCurrent();
    echo $current instanceof Fiber ? "In fiber\n" : "Not in fiber\n";
});
```

## Important Concepts

1. **Only one fiber runs at a time** - No true parallelism
2. **Explicit suspension required** - Code won't pause automatically
3. **Foundation for async libraries** - Not typically used directly
4. **Can throw exceptions** - Both from fiber and into fiber

## Code Examples

**Fibonacci sequence generator using Fiber**

```php
<?php
// Simple generator-like behavior with Fiber
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

// Collect first 10 Fibonacci numbers
$numbers[] = $fib->start();
while (!$fib->isTerminated()) {
    $numbers[] = $fib->resume();
}

array_pop($numbers); // Remove last null
print_r($numbers);
// [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]
?>
```


## Resources

- [Fibers Overview](https://www.php.net/manual/en/language.fibers.php) — Official PHP Fibers documentation

---

> 📘 *This lesson is part of the [Modern PHP 8.x: Latest Language Features](https://stanza.dev/courses/php-modern-features) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*