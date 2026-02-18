---
source_course: "php-async"
source_lesson: "php-async-reactphp-promises"
---

# Promises in ReactPHP

ReactPHP uses promises extensively for async operations. Understanding them is essential for effective async programming.

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

## Resources

- [ReactPHP Promise Documentation](https://reactphp.org/promise/) — Complete guide to ReactPHP promises

---

> 📘 *This lesson is part of the [Asynchronous PHP](https://stanza.dev/courses/php-async) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*