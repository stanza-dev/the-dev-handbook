---
source_course: "php-async"
source_lesson: "php-async-reactphp-introduction"
---

# Introduction to ReactPHP

ReactPHP is a low-level library for event-driven programming in PHP. It provides the foundational components for building non-blocking, asynchronous applications.

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

## Resources

- [ReactPHP Documentation](https://reactphp.org/) — Official ReactPHP documentation and guides

---

> 📘 *This lesson is part of the [Asynchronous PHP](https://stanza.dev/courses/php-async) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*