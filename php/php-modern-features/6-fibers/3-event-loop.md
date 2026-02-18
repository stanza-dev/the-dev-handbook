---
source_course: "php-modern-features"
source_lesson: "php-modern-features-fibers-event-loop"
---

# Building an Event Loop with Fibers

Understanding how to build an event loop helps demystify async PHP libraries.

## Simple Event Loop

```php
<?php
class EventLoop
{
    private array $fibers = [];
    private array $timers = [];
    private array $pending = [];
    
    public function defer(callable $callback): void
    {
        $this->pending[] = $callback;
    }
    
    public function delay(float $seconds, callable $callback): void
    {
        $this->timers[] = [
            'time' => microtime(true) + $seconds,
            'callback' => $callback,
        ];
    }
    
    public function async(callable $callback): void
    {
        $this->fibers[] = new Fiber($callback);
    }
    
    public function run(): void
    {
        // Start all fibers
        foreach ($this->fibers as $fiber) {
            if (!$fiber->isStarted()) {
                $fiber->start($this);
            }
        }
        
        while ($this->hasWork()) {
            // Process pending callbacks
            foreach ($this->pending as $key => $callback) {
                unset($this->pending[$key]);
                $callback();
            }
            
            // Check timers
            $now = microtime(true);
            foreach ($this->timers as $key => $timer) {
                if ($timer['time'] <= $now) {
                    unset($this->timers[$key]);
                    $timer['callback']();
                }
            }
            
            // Resume suspended fibers
            foreach ($this->fibers as $key => $fiber) {
                if ($fiber->isSuspended()) {
                    $fiber->resume();
                }
                if ($fiber->isTerminated()) {
                    unset($this->fibers[$key]);
                }
            }
            
            // Small delay to prevent CPU spinning
            usleep(1000);
        }
    }
    
    private function hasWork(): bool
    {
        return !empty($this->fibers) || 
               !empty($this->timers) || 
               !empty($this->pending);
    }
}
```

## Async Sleep Implementation

```php
<?php
function asyncSleep(EventLoop $loop, float $seconds): void
{
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

// Usage
$loop = new EventLoop();

$loop->async(function() use ($loop) {
    echo "Task 1: Start\n";
    asyncSleep($loop, 0.5);
    echo "Task 1: After 500ms\n";
    asyncSleep($loop, 0.5);
    echo "Task 1: Complete\n";
});

$loop->async(function() use ($loop) {
    echo "Task 2: Start\n";
    asyncSleep($loop, 0.3);
    echo "Task 2: After 300ms\n";
    asyncSleep($loop, 0.3);
    echo "Task 2: Complete\n";
});

$loop->run();
// Both tasks run concurrently!
```

## Promise-Like Pattern

```php
<?php
class Deferred
{
    private ?Fiber $waiting = null;
    private mixed $result = null;
    private bool $resolved = false;
    
    public function resolve(mixed $value): void
    {
        $this->result = $value;
        $this->resolved = true;
        
        if ($this->waiting?->isSuspended()) {
            $this->waiting->resume($value);
        }
    }
    
    public function await(): mixed
    {
        if ($this->resolved) {
            return $this->result;
        }
        
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

$fiber->start();
// Later...
$deferred->resolve('Hello!');
// Output: Got: Hello!
```

## Resources

- [Fibers](https://www.php.net/manual/en/language.fibers.php) — PHP Fibers documentation

---

> 📘 *This lesson is part of the [Modern PHP 8.x: Latest Language Features](https://stanza.dev/courses/php-modern-features) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*