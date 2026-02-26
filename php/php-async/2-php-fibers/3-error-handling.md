---
source_course: "php-async"
source_lesson: "php-async-fiber-error-handling"
---

# Error Handling in Fibers

## Introduction
Proper error handling is crucial when working with Fibers because exceptions can propagate in unexpected ways between the Fiber and its caller. In long-running async applications, a single unhandled exception can crash the entire server, making robust error handling essential.
## Key Concepts
- **Exception Propagation**: Exceptions thrown inside a Fiber propagate to the caller of start() or resume(), where they can be caught normally.
- **Fiber::throw()**: A method to inject an exception into a suspended Fiber at its suspension point, useful for signaling timeouts or cancellations.
- **Supervisor Pattern**: An error handling strategy where a supervisor manages Fiber execution, implementing retry logic for retryable failures.
- **AsyncResult**: A Result type pattern (success/failure wrapper) that makes error handling explicit and composable for async operations.
## Real World Context
Libraries like ReactPHP and Amp use Fibers internally to provide clean async/await-style APIs. Understanding how Fibers work helps you debug issues, write custom async utilities, and make informed architectural decisions.

## Deep Dive
## Exceptions Inside Fibers

Exceptions thrown inside a Fiber propagate to the caller of `start()` or `resume()`:

```php
<?php
$fiber = new Fiber(function(): void {
    echo "Before exception\n";
    throw new RuntimeException('Fiber error!');
    echo "After exception\n";  // Never reached
});

try {
    $fiber->start();
} catch (RuntimeException $e) {
    echo "Caught: " . $e->getMessage() . "\n";
}

echo "Fiber terminated: " . ($fiber->isTerminated() ? 'yes' : 'no') . "\n";
```

Output:
```
Before exception
Caught: Fiber error!
Fiber terminated: yes
```

## Throwing Into Fibers

You can inject exceptions into suspended Fibers:

```php
<?php
$fiber = new Fiber(function(): string {
    try {
        echo "Waiting for data...\n";
        $data = Fiber::suspend();
        return "Received: {$data}";
    } catch (TimeoutException $e) {
        return "Timeout: " . $e->getMessage();
    }
});

$fiber->start();

// Simulate a timeout condition
$timedOut = true;

if ($timedOut) {
    $fiber->throw(new TimeoutException('Request took too long'));
} else {
    $fiber->resume('some data');
}

echo $fiber->getReturn() . "\n";
```

## Error Handling Pattern: Supervisors

```php
<?php
class FiberSupervisor {
    /** @var array<string, Fiber> */
    private array $fibers = [];
    
    /** @var array<string, array{attempts: int, maxRetries: int}> */
    private array $retryInfo = [];
    
    public function spawn(
        string $id, 
        callable $task, 
        int $maxRetries = 3
    ): void {
        $this->fibers[$id] = new Fiber($task);
        $this->retryInfo[$id] = [
            'attempts' => 0,
            'maxRetries' => $maxRetries,
            'task' => $task
        ];
    }
    
    public function run(): array {
        $results = [];
        $errors = [];
        
        foreach ($this->fibers as $id => $fiber) {
            try {
                $results[$id] = $this->runFiber($id, $fiber);
            } catch (Throwable $e) {
                $errors[$id] = $e->getMessage();
            }
        }
        
        return ['results' => $results, 'errors' => $errors];
    }
    
    private function runFiber(string $id, Fiber $fiber): mixed {
        $info = $this->retryInfo[$id];
        
        while ($info['attempts'] < $info['maxRetries']) {
            try {
                if (!$fiber->isStarted()) {
                    $fiber->start();
                }
                
                while (!$fiber->isTerminated()) {
                    if ($fiber->isSuspended()) {
                        $fiber->resume();
                    }
                }
                
                return $fiber->getReturn();
                
            } catch (RetryableException $e) {
                $info['attempts']++;
                $this->retryInfo[$id] = $info;
                
                echo "[{$id}] Retry {$info['attempts']}/{$info['maxRetries']}\n";
                
                // Create new fiber for retry
                $fiber = new Fiber($info['task']);
                
            } catch (Throwable $e) {
                // Non-retryable error
                throw $e;
            }
        }
        
        throw new MaxRetriesException("Max retries exceeded for {$id}");
    }
}
```

## Finally Blocks and Cleanup

```php
<?php
$fiber = new Fiber(function(): void {
    $resource = fopen('file.txt', 'r');
    
    try {
        while (!feof($resource)) {
            $line = fgets($resource);
            Fiber::suspend($line);
        }
    } finally {
        // Always executed, even if fiber is terminated early
        fclose($resource);
        echo "Resource cleaned up\n";
    }
});

$fiber->start();
$fiber->resume();  // Read next line

// Terminate early - finally still runs
$fiber->throw(new Exception('Early termination'));
```

## Error Propagation Best Practices

```php
<?php
class AsyncResult {
    public function __construct(
        public readonly bool $success,
        public readonly mixed $value = null,
        public readonly ?Throwable $error = null
    ) {}
    
    public static function ok(mixed $value): self {
        return new self(true, $value);
    }
    
    public static function fail(Throwable $error): self {
        return new self(false, null, $error);
    }
    
    public function unwrap(): mixed {
        if (!$this->success) {
            throw $this->error ?? new RuntimeException('Unknown error');
        }
        return $this->value;
    }
}

function safeRunFiber(Fiber $fiber): AsyncResult {
    try {
        $fiber->start();
        
        while ($fiber->isSuspended()) {
            $fiber->resume();
        }
        
        return AsyncResult::ok($fiber->getReturn());
        
    } catch (Throwable $e) {
        return AsyncResult::fail($e);
    }
}

// Usage
$result = safeRunFiber(new Fiber(fn() => riskyOperation()));

if ($result->success) {
    echo "Success: " . $result->value . "\n";
} else {
    echo "Failed: " . $result->error->getMessage() . "\n";
}
```

## Common Pitfalls
### 1. Forgetting to Check State

```php
<?php
// WRONG: May throw FiberError
$fiber->resume($data);  // What if fiber isn't suspended?

// CORRECT: Always check state
if ($fiber->isSuspended()) {
    $fiber->resume($data);
}
```

### 2. Unhandled Exceptions in Async Context

```php
<?php
// WRONG: Exception may be lost
$scheduler->spawn(function() {
    throw new Exception('Oops');
});

// CORRECT: Handle at spawn point
$scheduler->spawn(function() {
    try {
        riskyOperation();
    } catch (Exception $e) {
        logError($e);
        // Re-throw if needed
    }
});
```

### 3. Deadlocks with Suspend

```php
<?php
// WRONG: Fiber never resumed
$fiber = new Fiber(function() {
    Fiber::suspend();  // Who will resume this?
    return 'done';
});
$fiber->start();
// Fiber is suspended forever

// CORRECT: Ensure resume path exists
// Use a scheduler or event loop that tracks suspended fibers
```

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

- [PHP Manual - Fiber Error Handling](https://www.php.net/manual/en/class.fibererror.php) — Documentation on FiberError exceptions

---

> 📘 *This lesson is part of the [Asynchronous PHP](https://stanza.dev/courses/php-async) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*