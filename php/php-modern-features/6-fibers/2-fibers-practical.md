---
source_course: "php-modern-features"
source_lesson: "php-modern-features-fibers-practical"
---

# Practical Fiber Patterns

While you rarely use Fibers directly, understanding patterns helps when working with async libraries.

## Simple Scheduler

```php
<?php
class SimpleScheduler {
    private array $fibers = [];
    
    public function add(callable $task): void {
        $this->fibers[] = new Fiber($task);
    }
    
    public function run(): void {
        // Start all fibers
        foreach ($this->fibers as $fiber) {
            $fiber->start();
        }
        
        // Keep running until all complete
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

## Async Sleep Simulation

```php
<?php
// In real async libraries, sleep doesn't block
class AsyncRuntime {
    private array $sleeping = [];
    
    public function sleep(float $seconds): void {
        $fiber = Fiber::getCurrent();
        $wakeAt = microtime(true) + $seconds;
        
        $this->sleeping[] = [
            'fiber' => $fiber,
            'wakeAt' => $wakeAt,
        ];
        
        Fiber::suspend();
    }
    
    public function tick(): void {
        $now = microtime(true);
        
        foreach ($this->sleeping as $key => $item) {
            if ($item['wakeAt'] <= $now) {
                unset($this->sleeping[$key]);
                $item['fiber']->resume();
            }
        }
    }
}
```

## Exception Handling

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
$result = $fiber->throw(new Exception('Something went wrong'));
echo $result;  // 'handled'
```

## Real-World Usage

Fibers power async libraries like:

- **ReactPHP** - Event-driven programming
- **Amp** - Async framework
- **Revolt** - Event loop implementation

```php
<?php
// How async libraries use Fibers (conceptual)

// User writes this:
async function fetchData() {
    $response = await httpGet('https://api.example.com');
    return $response->body;
}

// Library translates to:
function fetchData() {
    $fiber = Fiber::getCurrent();
    
    httpGetAsync('https://api.example.com', function($response) use ($fiber) {
        $fiber->resume($response);
    });
    
    return Fiber::suspend();
}
```

## When to Use Fibers

1. **Building async frameworks** - Foundation for event loops
2. **Cooperative multitasking** - Multiple concurrent operations
3. **Coroutine-like patterns** - Pausable computations

**Usually, you'll use libraries built on Fibers rather than Fibers directly.**

## Code Examples

**Concurrent task runner with Fibers**

```php
<?php
// Concurrent task runner using Fibers
class TaskRunner {
    private array $tasks = [];
    private array $results = [];
    
    public function addTask(string $name, callable $task): self {
        $this->tasks[$name] = new Fiber($task);
        return $this;
    }
    
    public function run(): array {
        // Start all tasks
        foreach ($this->tasks as $name => $fiber) {
            echo "Starting: $name\n";
            $fiber->start();
        }
        
        // Process until all complete
        $pending = count($this->tasks);
        
        while ($pending > 0) {
            foreach ($this->tasks as $name => $fiber) {
                if ($fiber->isSuspended()) {
                    $fiber->resume();
                }
                
                if ($fiber->isTerminated() && !isset($this->results[$name])) {
                    $this->results[$name] = $fiber->getReturn();
                    $pending--;
                    echo "Completed: $name\n";
                }
            }
        }
        
        return $this->results;
    }
}

// Usage
$runner = new TaskRunner();

$runner->addTask('task1', function() {
    for ($i = 0; $i < 3; $i++) {
        echo "Task 1 step $i\n";
        Fiber::suspend();
    }
    return 'Task 1 done';
});

$runner->addTask('task2', function() {
    for ($i = 0; $i < 2; $i++) {
        echo "Task 2 step $i\n";
        Fiber::suspend();
    }
    return 'Task 2 done';
});

$results = $runner->run();
print_r($results);
?>
```


## Resources

- [Fiber Class](https://www.php.net/manual/en/class.fiber.php) — Complete Fiber class reference

---

> 📘 *This lesson is part of the [Modern PHP 8.x: Latest Language Features](https://stanza.dev/courses/php-modern-features) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*