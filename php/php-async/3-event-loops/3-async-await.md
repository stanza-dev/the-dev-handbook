---
source_course: "php-async"
source_lesson: "php-async-promises-async-await"
---

# Promises and Async/Await Patterns

Promises provide a cleaner way to handle asynchronous operations than callbacks. Combined with Fibers, PHP can support async/await-style syntax.

## What is a Promise?

A **Promise** represents a value that may not be available yet but will be at some point, or will fail with an error.

```php
<?php
// Promise states
enum PromiseState {
    case Pending;   // Not yet resolved
    case Fulfilled; // Successfully completed with a value
    case Rejected;  // Failed with an error
}

class Promise {
    private PromiseState $state = PromiseState::Pending;
    private mixed $value = null;
    private ?Throwable $reason = null;
    private array $onFulfilled = [];
    private array $onRejected = [];
    
    public function then(
        ?callable $onFulfilled = null,
        ?callable $onRejected = null
    ): self {
        $next = new self();
        
        $handleFulfilled = function($value) use ($onFulfilled, $next) {
            try {
                if ($onFulfilled === null) {
                    $next->resolve($value);
                } else {
                    $result = $onFulfilled($value);
                    if ($result instanceof self) {
                        $result->then(
                            fn($v) => $next->resolve($v),
                            fn($e) => $next->reject($e)
                        );
                    } else {
                        $next->resolve($result);
                    }
                }
            } catch (Throwable $e) {
                $next->reject($e);
            }
        };
        
        $handleRejected = function($reason) use ($onRejected, $next) {
            try {
                if ($onRejected === null) {
                    $next->reject($reason);
                } else {
                    $result = $onRejected($reason);
                    $next->resolve($result);
                }
            } catch (Throwable $e) {
                $next->reject($e);
            }
        };
        
        match ($this->state) {
            PromiseState::Pending => (
                $this->onFulfilled[] = $handleFulfilled,
                $this->onRejected[] = $handleRejected
            ),
            PromiseState::Fulfilled => $handleFulfilled($this->value),
            PromiseState::Rejected => $handleRejected($this->reason),
        };
        
        return $next;
    }
    
    public function catch(callable $onRejected): self {
        return $this->then(null, $onRejected);
    }
    
    public function finally(callable $callback): self {
        return $this->then(
            function($value) use ($callback) {
                $callback();
                return $value;
            },
            function($reason) use ($callback) {
                $callback();
                throw $reason;
            }
        );
    }
    
    public function resolve(mixed $value): void {
        if ($this->state !== PromiseState::Pending) return;
        
        $this->state = PromiseState::Fulfilled;
        $this->value = $value;
        
        foreach ($this->onFulfilled as $callback) {
            $callback($value);
        }
    }
    
    public function reject(Throwable $reason): void {
        if ($this->state !== PromiseState::Pending) return;
        
        $this->state = PromiseState::Rejected;
        $this->reason = $reason;
        
        foreach ($this->onRejected as $callback) {
            $callback($reason);
        }
    }
}
```

## Using Promises

```php
<?php
// Create a promise that resolves after a delay
function delay(float $seconds): Promise {
    $promise = new Promise();
    
    // In real code, this would use an event loop
    // For demo, we'll resolve immediately
    $promise->resolve($seconds);
    
    return $promise;
}

// Chain promises
delay(1.0)
    ->then(function($seconds) {
        echo "Waited {$seconds} seconds\n";
        return fetchUser(1);  // Returns another Promise
    })
    ->then(function($user) {
        echo "Got user: {$user['name']}\n";
        return fetchOrders($user['id']);
    })
    ->then(function($orders) {
        echo "Got " . count($orders) . " orders\n";
    })
    ->catch(function($error) {
        echo "Error: " . $error->getMessage() . "\n";
    })
    ->finally(function() {
        echo "Cleanup complete\n";
    });
```

## Promise Combinators

```php
<?php
class PromiseUtils {
    /**
     * Wait for all promises to resolve
     * Rejects if any promise rejects
     */
    public static function all(array $promises): Promise {
        $result = new Promise();
        $values = [];
        $remaining = count($promises);
        
        if ($remaining === 0) {
            $result->resolve([]);
            return $result;
        }
        
        foreach ($promises as $key => $promise) {
            $promise->then(
                function($value) use ($key, &$values, &$remaining, $result) {
                    $values[$key] = $value;
                    if (--$remaining === 0) {
                        ksort($values);
                        $result->resolve($values);
                    }
                },
                fn($error) => $result->reject($error)
            );
        }
        
        return $result;
    }
    
    /**
     * Wait for first promise to settle (resolve or reject)
     */
    public static function race(array $promises): Promise {
        $result = new Promise();
        
        foreach ($promises as $promise) {
            $promise->then(
                fn($value) => $result->resolve($value),
                fn($error) => $result->reject($error)
            );
        }
        
        return $result;
    }
    
    /**
     * Wait for all promises to settle (never rejects)
     */
    public static function allSettled(array $promises): Promise {
        $result = new Promise();
        $outcomes = [];
        $remaining = count($promises);
        
        if ($remaining === 0) {
            $result->resolve([]);
            return $result;
        }
        
        foreach ($promises as $key => $promise) {
            $promise->then(
                function($value) use ($key, &$outcomes, &$remaining, $result) {
                    $outcomes[$key] = ['status' => 'fulfilled', 'value' => $value];
                    if (--$remaining === 0) {
                        ksort($outcomes);
                        $result->resolve($outcomes);
                    }
                },
                function($error) use ($key, &$outcomes, &$remaining, $result) {
                    $outcomes[$key] = ['status' => 'rejected', 'reason' => $error];
                    if (--$remaining === 0) {
                        ksort($outcomes);
                        $result->resolve($outcomes);
                    }
                }
            );
        }
        
        return $result;
    }
}

// Usage
$results = PromiseUtils::all([
    fetchUser(1),
    fetchUser(2),
    fetchUser(3),
])->then(function($users) {
    foreach ($users as $user) {
        echo "User: {$user['name']}\n";
    }
});
```

## Async/Await with Fibers

```php
<?php
class AsyncRuntime {
    private static ?self $instance = null;
    private array $pending = [];
    
    public static function getInstance(): self {
        return self::$instance ??= new self();
    }
    
    public function await(Promise $promise): mixed {
        $fiber = Fiber::getCurrent();
        
        if ($fiber === null) {
            throw new LogicException('await must be called inside async context');
        }
        
        $resolved = false;
        $result = null;
        $error = null;
        
        $promise->then(
            function($value) use (&$resolved, &$result, $fiber) {
                $resolved = true;
                $result = $value;
                if ($fiber->isSuspended()) {
                    $this->pending[] = fn() => $fiber->resume($value);
                }
            },
            function($e) use (&$resolved, &$error, $fiber) {
                $resolved = true;
                $error = $e;
                if ($fiber->isSuspended()) {
                    $this->pending[] = fn() => $fiber->throw($e);
                }
            }
        );
        
        if ($resolved) {
            if ($error) throw $error;
            return $result;
        }
        
        return Fiber::suspend();
    }
    
    public function async(callable $fn): Promise {
        $promise = new Promise();
        $fiber = new Fiber($fn);
        
        try {
            $result = $fiber->start();
            
            if ($fiber->isTerminated()) {
                $promise->resolve($fiber->getReturn());
            }
        } catch (Throwable $e) {
            $promise->reject($e);
        }
        
        return $promise;
    }
    
    public function run(): void {
        while (!empty($this->pending)) {
            $callback = array_shift($this->pending);
            $callback();
        }
    }
}

// Helper functions
function async(callable $fn): Promise {
    return AsyncRuntime::getInstance()->async($fn);
}

function await(Promise $promise): mixed {
    return AsyncRuntime::getInstance()->await($promise);
}

// Usage - looks like synchronous code!
async(function() {
    echo "Starting...\n";
    
    $user = await(fetchUserAsync(1));
    echo "Got user: {$user['name']}\n";
    
    $orders = await(fetchOrdersAsync($user['id']));
    echo "Got " . count($orders) . " orders\n";
    
    return $orders;
});
```

## Real-World Example: Parallel API Calls

```php
<?php
async(function() {
    // Start all requests at once
    $userPromise = fetchUserAsync(1);
    $ordersPromise = fetchOrdersAsync(1);
    $recommendationsPromise = fetchRecommendationsAsync(1);
    
    // Wait for all to complete
    $results = await(PromiseUtils::all([
        'user' => $userPromise,
        'orders' => $ordersPromise,
        'recommendations' => $recommendationsPromise,
    ]));
    
    return $results;
});
```

## Resources

- [Guzzle Promises](https://github.com/guzzle/promises) — Production-ready Promise implementation for PHP

---

> 📘 *This lesson is part of the [Asynchronous PHP](https://stanza.dev/courses/php-async) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*