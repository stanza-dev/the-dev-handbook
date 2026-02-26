---
source_course: "php-async"
source_lesson: "php-async-reactphp-introduction"
---

# Introduction to ReactPHP

## Introduction
ReactPHP is a low-level library for event-driven programming in PHP that provides the foundational components for building non-blocking, asynchronous applications. It is not a framework but a collection of composable components that power many production PHP async applications.
## Key Concepts
- **ReactPHP**: A low-level library collection for event-driven programming in PHP, providing components for the event loop, streams, promises, HTTP, and more.
- **Loop (Event Loop)**: ReactPHP's core component that drives all async operations, managing timers, stream I/O, and deferred callbacks.
- **Browser**: ReactPHP's async HTTP client that returns Promises for non-blocking HTTP requests with automatic connection pooling.
- **HttpServer**: A non-blocking HTTP server that can handle many concurrent connections from a single PHP process.
- **futureTick()**: Schedules a callback to run on the very next iteration of the event loop, before any timers or I/O checks.
## Real World Context
ReactPHP powers production WebSocket servers, API gateways, and real-time dashboards. Companies use it to handle tens of thousands of concurrent connections from a single PHP process, making it a practical alternative to Node.js for event-driven applications.

## Deep Dive
## What is ReactPHP?

ReactPHP is not a framework—it's a collection of components that work together:

- **EventLoop**: The core component that drives everything
- **Stream**: Non-blocking I/O streams
- **Promise**: Async result handling
- **Socket**: TCP/UDP client and server
- **HTTP**: Async HTTP client and server
- **DNS**: Async DNS resolver
- **Child Process**: Non-blocking process execution

## Installation

```bash
composer require react/event-loop react/http react/socket
```

## The Event Loop

The event loop is ReactPHP's core. All async operations are registered with it:

```php
<?php
require 'vendor/autoload.php';

use React\EventLoop\Loop;

// Add a timer
Loop::addTimer(2.0, function () {
    echo "2 seconds have passed!\n";
});

// Add a periodic timer
$counter = 0;
Loop::addPeriodicTimer(1.0, function () use (&$counter) {
    $counter++;
    echo "Tick {$counter}\n";
    
    if ($counter >= 5) {
        Loop::stop();
    }
});

echo "Starting event loop...\n";

// This blocks until there's no more work
Loop::run();

echo "Event loop finished\n";
```

Output:
```
Starting event loop...
Tick 1
Tick 2
2 seconds have passed!
Tick 3
Tick 4
Tick 5
Event loop finished
```

## Event Loop Methods

| Method | Description |
|--------|-------------|
| `Loop::addTimer($seconds, $callback)` | Run callback once after delay |
| `Loop::addPeriodicTimer($interval, $callback)` | Run callback repeatedly |
| `Loop::cancelTimer($timer)` | Cancel a scheduled timer |
| `Loop::addReadStream($stream, $callback)` | Watch stream for reading |
| `Loop::addWriteStream($stream, $callback)` | Watch stream for writing |
| `Loop::removeReadStream($stream)` | Stop watching for reads |
| `Loop::removeWriteStream($stream)` | Stop watching for writes |
| `Loop::futureTick($callback)` | Run callback on next tick |
| `Loop::run()` | Start the event loop |
| `Loop::stop()` | Stop the event loop |

## Async HTTP Client

```php
<?php
require 'vendor/autoload.php';

use React\Http\Browser;
use React\EventLoop\Loop;

$browser = new Browser();

// Single request
$browser->get('https://httpbin.org/get')
    ->then(function (Psr\Http\Message\ResponseInterface $response) {
        echo 'Status: ' . $response->getStatusCode() . "\n";
        echo 'Body: ' . $response->getBody() . "\n";
    })
    ->catch(function (Exception $e) {
        echo 'Error: ' . $e->getMessage() . "\n";
    });

echo "Request sent (not blocking)\n";

// Loop runs automatically with ReactPHP 1.x
```

## Concurrent Requests

```php
<?php
require 'vendor/autoload.php';

use React\Http\Browser;
use React\Promise\Utils;

$browser = new Browser();

$urls = [
    'https://httpbin.org/delay/2',
    'https://httpbin.org/delay/1',
    'https://httpbin.org/delay/3',
];

$startTime = microtime(true);

// Start all requests concurrently
$promises = array_map(
    fn($url) => $browser->get($url),
    $urls
);

// Wait for all to complete
Utils::all($promises)
    ->then(function ($responses) use ($startTime) {
        $elapsed = microtime(true) - $startTime;
        
        echo "All " . count($responses) . " requests completed\n";
        echo "Total time: " . round($elapsed, 2) . "s\n";
        // With sequential: ~6 seconds
        // With concurrent: ~3 seconds (longest delay)
    })
    ->catch(function (Exception $e) {
        echo "Error: " . $e->getMessage() . "\n";
    });
```

## Creating an HTTP Server

```php
<?php
require 'vendor/autoload.php';

use React\Http\HttpServer;
use React\Http\Message\Response;
use React\Socket\SocketServer;
use Psr\Http\Message\ServerRequestInterface;

$server = new HttpServer(function (ServerRequestInterface $request) {
    $method = $request->getMethod();
    $path = $request->getUri()->getPath();
    
    return match (true) {
        $path === '/' => new Response(
            200,
            ['Content-Type' => 'text/html'],
            '<h1>Welcome to ReactPHP!</h1>'
        ),
        $path === '/api/time' => new Response(
            200,
            ['Content-Type' => 'application/json'],
            json_encode(['time' => date('c')])
        ),
        default => new Response(
            404,
            ['Content-Type' => 'text/plain'],
            'Not Found'
        )
    };
});

$socket = new SocketServer('0.0.0.0:8080');
$server->listen($socket);

echo "Server running at http://localhost:8080\n";
```

## Async Database Queries

```php
<?php
require 'vendor/autoload.php';

use React\MySQL\Factory;
use React\MySQL\QueryResult;

$factory = new Factory();

$connection = $factory->createLazyConnection('user:pass@localhost/database');

$connection->query('SELECT * FROM users WHERE active = ?', [1])
    ->then(function (QueryResult $result) {
        echo "Found " . count($result->resultRows) . " users:\n";
        foreach ($result->resultRows as $row) {
            echo "- {$row['name']}\n";
        }
    })
    ->catch(function (Exception $e) {
        echo "Query failed: " . $e->getMessage() . "\n";
    })
    ->finally(function () use ($connection) {
        $connection->quit();
    });
```

## Common Pitfalls
1. **Calling Loop::run() multiple times** - The event loop should only be started once. Multiple run() calls indicate architectural issues.
2. **Not handling connection errors** - HTTP requests can fail for many reasons. Always add catch() handlers to browser requests.
3. **Forgetting that Loop::run() blocks** - Code after Loop::run() only executes after the loop stops. Structure your program flow accordingly.

## Best Practices
1. **Use the Browser class for HTTP requests** - ReactPHP's Browser provides a simple, promise-based API for async HTTP that handles connection pooling automatically.
2. **Leverage promise combinators** - Use Utils::all() for parallel requests, Utils::race() for fastest-wins, and Utils::any() for first-success patterns.
3. **Handle backpressure in streams** - When piping streams, use pipe() which handles backpressure automatically, or manually pause/resume based on write() return values.

## Summary
- ReactPHP is a collection of components for event-driven PHP, not a monolithic framework.
- The EventLoop drives all async operations: timers, streams, and I/O.
- Browser provides a simple async HTTP client with promise-based responses.
- HttpServer enables building high-performance, non-blocking web servers.
- ReactPHP supports async database queries, DNS resolution, and child process management.

## Resources

- [ReactPHP Documentation](https://reactphp.org/) — Official ReactPHP documentation and guides

---

> 📘 *This lesson is part of the [Asynchronous PHP](https://stanza.dev/courses/php-async) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*