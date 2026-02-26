---
source_course: "php-async"
source_lesson: "php-async-reactphp-promises"
---

# Promises in ReactPHP

## Introduction
ReactPHP uses promises extensively for handling async operations. Understanding how to create, chain, combine, and cancel promises is essential for writing effective async code with the ReactPHP ecosystem.
## Key Concepts
- **Deferred**: A ReactPHP class that creates a Promise you can resolve or reject externally, useful for wrapping callback-based APIs.
- **Utils::all()**: Waits for all promises to resolve and returns an array of results, rejecting immediately if any promise fails.
- **Utils::race()**: Returns the result of the first promise to settle (resolve or reject), regardless of the others.
- **Cancellation**: The ability to cancel a pending promise, cleaning up associated resources like timers or network connections.
- **Error Recovery**: Using the second parameter of then() or catch() to handle specific errors and continue the promise chain with a fallback value.
## Real World Context
Promises are the primary abstraction for handling async results in ReactPHP and Guzzle. Whether you are fetching data from APIs, querying databases, or reading files, promises provide a consistent interface for managing async operations.

## Deep Dive
## Creating Promises

```php
<?php
require 'vendor/autoload.php';

use React\Promise\Deferred;
use React\Promise\Promise;

// Using Deferred
function fetchDataAsync(): Promise {
    $deferred = new Deferred();
    
    // Simulate async operation
    React\EventLoop\Loop::addTimer(1.0, function () use ($deferred) {
        $deferred->resolve(['data' => 'fetched']);
    });
    
    return $deferred->promise();
}

// Using Promise constructor
function fetchData2(): Promise {
    return new Promise(function ($resolve, $reject) {
        React\EventLoop\Loop::addTimer(1.0, function () use ($resolve) {
            $resolve(['data' => 'fetched']);
        });
    });
}

// Static helpers
$resolved = React\Promise\resolve('immediate value');
$rejected = React\Promise\reject(new Exception('error'));
```

## Promise Chaining

```php
<?php
use React\Promise\Promise;

fetchUser($userId)
    ->then(function ($user) {
        echo "Got user: {$user['name']}\n";
        return fetchOrders($user['id']);  // Return promise for chaining
    })
    ->then(function ($orders) {
        echo "Got " . count($orders) . " orders\n";
        return processOrders($orders);
    })
    ->then(function ($result) {
        echo "Processing complete\n";
    })
    ->catch(function (Exception $e) {
        echo "Error: " . $e->getMessage() . "\n";
    })
    ->finally(function () {
        echo "Cleanup\n";
    });
```

## Promise Combinators

```php
<?php
use React\Promise\Utils;

// all() - Wait for all promises
$promises = [
    fetchUser(1),
    fetchUser(2),
    fetchUser(3),
];

Utils::all($promises)
    ->then(function ($users) {
        // $users is array of all results
        foreach ($users as $user) {
            echo "User: {$user['name']}\n";
        }
    })
    ->catch(function ($e) {
        // Called if ANY promise rejects
        echo "Failed: " . $e->getMessage() . "\n";
    });

// race() - First to settle wins
Utils::race([
    fetchFromServer1(),
    fetchFromServer2(),
    fetchFromServer3(),
])->then(function ($result) {
    echo "First response: " . json_encode($result) . "\n";
});

// any() - First to resolve wins (ignores rejections)
Utils::any([
    fetchFromServer1(),  // Might fail
    fetchFromServer2(),  // Might fail
    fetchFromServer3(),  // Might succeed
])->then(function ($result) {
    echo "First success: " . json_encode($result) . "\n";
});
```

## Error Handling Patterns

```php
<?php
use React\Promise\Promise;

// Per-step error handling
fetchUser($userId)
    ->then(
        function ($user) {
            return fetchOrders($user['id']);
        },
        function ($e) {
            // Handle user fetch error specifically
            echo "User not found, using default\n";
            return fetchOrders(0);  // Recover with default
        }
    )
    ->then(function ($orders) {
        return processOrders($orders);
    })
    ->catch(function ($e) {
        // Catch any remaining errors
        echo "Error: " . $e->getMessage() . "\n";
    });

// Retry pattern
function withRetry(callable $operation, int $maxRetries = 3): Promise {
    return new Promise(function ($resolve, $reject) use ($operation, $maxRetries) {
        $attempt = function ($remainingRetries) use (&$attempt, $operation, $resolve, $reject) {
            $operation()
                ->then($resolve)
                ->catch(function ($e) use ($attempt, $remainingRetries, $reject) {
                    if ($remainingRetries > 0) {
                        echo "Retrying... ({$remainingRetries} left)\n";
                        React\EventLoop\Loop::addTimer(1.0, function () use ($attempt, $remainingRetries) {
                            $attempt($remainingRetries - 1);
                        });
                    } else {
                        $reject($e);
                    }
                });
        };
        
        $attempt($maxRetries);
    });
}

// Timeout pattern
function withTimeout(Promise $promise, float $seconds): Promise {
    return Utils::race([
        $promise,
        new Promise(function ($resolve, $reject) use ($seconds) {
            React\EventLoop\Loop::addTimer($seconds, function () use ($reject) {
                $reject(new TimeoutException("Operation timed out"));
            });
        })
    ]);
}

// Usage
withTimeout(slowOperation(), 5.0)
    ->then(fn($result) => echo "Success\n")
    ->catch(fn($e) => echo "Timed out or failed\n");
```

## Converting Callbacks to Promises

```php
<?php
use React\Promise\Deferred;

// Wrap callback-based API
function readFileAsync(string $path): Promise {
    $deferred = new Deferred();
    
    $stream = new React\Stream\ReadableResourceStream(
        fopen($path, 'r')
    );
    
    $content = '';
    
    $stream->on('data', function ($chunk) use (&$content) {
        $content .= $chunk;
    });
    
    $stream->on('end', function () use ($deferred, &$content) {
        $deferred->resolve($content);
    });
    
    $stream->on('error', function ($e) use ($deferred) {
        $deferred->reject($e);
    });
    
    return $deferred->promise();
}

// Usage
readFileAsync('large-file.txt')
    ->then(function ($content) {
        echo "File size: " . strlen($content) . " bytes\n";
    });
```

## Cancellation

```php
<?php
use React\Promise\Promise;
use React\Promise\CancellablePromiseInterface;

function cancellableOperation(): CancellablePromiseInterface {
    $timer = null;
    
    return new Promise(
        function ($resolve, $reject) use (&$timer) {
            $timer = React\EventLoop\Loop::addTimer(10.0, function () use ($resolve) {
                $resolve('completed');
            });
        },
        function () use (&$timer) {
            // Canceller function
            if ($timer !== null) {
                React\EventLoop\Loop::cancelTimer($timer);
            }
        }
    );
}

$promise = cancellableOperation();

// Cancel after 2 seconds
React\EventLoop\Loop::addTimer(2.0, function () use ($promise) {
    $promise->cancel();
    echo "Operation cancelled\n";
});
```

## Common Pitfalls
1. **Forgetting to handle rejections** - Unhandled promise rejections can silently swallow errors. Always add a catch() handler.
2. **Creating promise chains without returning** - In then() callbacks, you must return the next promise for proper chaining. Forgetting this breaks the chain.
3. **Using synchronous blocking inside then() callbacks** - This blocks the event loop, defeating the purpose of promises.

## Best Practices
1. **Always add error handlers** - Every promise chain should end with catch() to handle rejections. Unhandled rejections are silent bugs.
2. **Return promises from then() callbacks** - To properly chain async operations, return the next promise from your then() callback.
3. **Use finally() for cleanup** - Cleanup operations like closing connections should go in finally() so they run regardless of success or failure.

## Summary
- Promises represent eventual values that may resolve (success) or reject (failure).
- Promise chaining with then() creates readable sequential async flows.
- Utils::all() runs promises concurrently and collects all results.
- Utils::race() returns the first promise to settle; Utils::any() returns the first to resolve.
- Always handle rejections with catch() and use finally() for cleanup.

## Resources

- [ReactPHP Promise Documentation](https://reactphp.org/promise/) — Complete guide to ReactPHP promises

---

> 📘 *This lesson is part of the [Asynchronous PHP](https://stanza.dev/courses/php-async) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*