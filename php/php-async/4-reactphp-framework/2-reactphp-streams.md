---
source_course: "php-async"
source_lesson: "php-async-reactphp-streams"
---

# Working with ReactPHP Streams

## Introduction
Streams are the backbone of I/O in ReactPHP, providing a consistent event-driven interface for reading and writing data asynchronously. Whether you are handling file uploads, HTTP requests, or WebSocket frames, streams are the underlying abstraction.
## Key Concepts
- **ReadableStreamInterface**: A stream that emits 'data', 'end', 'error', and 'close' events as data becomes available.
- **WritableStreamInterface**: A stream that accepts data via write() and signals when its buffer is full or drained.
- **DuplexStreamInterface**: A stream that combines both readable and writable capabilities for bidirectional communication.
- **pipe()**: A method that connects a readable stream to a writable stream, automatically handling data transfer and back pressure.
- **ThroughStream**: A transform stream that processes data passing through it, enabling composable stream pipelines.
## Real World Context
PHP's stream functions power everything from file uploads to WebSocket servers. In production, non-blocking streams enable a single PHP process to handle thousands of simultaneous connections, which is essential for real-time features.

## Deep Dive
## Stream Interfaces

ReactPHP defines several stream interfaces:

| Interface | Description |
|-----------|-------------|
| `ReadableStreamInterface` | Can emit data events |
| `WritableStreamInterface` | Can receive data via write() |
| `DuplexStreamInterface` | Both readable and writable |

## Reading from Streams

```php
<?php
require 'vendor/autoload.php';

use React\Stream\ReadableResourceStream;
use React\EventLoop\Loop;

// Create readable stream from STDIN
$stream = new ReadableResourceStream(STDIN);

echo "Type something (Ctrl+D to end):\n";

// Handle incoming data
$stream->on('data', function (string $data) {
    echo "Received: " . trim($data) . "\n";
});

// Handle end of stream
$stream->on('end', function () {
    echo "Stream ended\n";
});

// Handle errors
$stream->on('error', function (Exception $e) {
    echo "Error: " . $e->getMessage() . "\n";
});

// Handle close
$stream->on('close', function () {
    echo "Stream closed\n";
});
```

## Writing to Streams

```php
<?php
require 'vendor/autoload.php';

use React\Stream\WritableResourceStream;
use React\EventLoop\Loop;

// Create writable stream to a file
$file = fopen('output.txt', 'w');
$stream = new WritableResourceStream($file);

// Write data
$stream->write("Line 1\n");
$stream->write("Line 2\n");
$stream->write("Line 3\n");

// End the stream (writes final data and closes)
$stream->end("Final line\n");

$stream->on('drain', function () {
    echo "Buffer drained, ready for more data\n";
});

$stream->on('close', function () {
    echo "File written and closed\n";
});
```

## Duplex Streams

Duplex streams combine reading and writing:

```php
<?php
require 'vendor/autoload.php';

use React\Stream\DuplexResourceStream;
use React\Socket\Connector;

$connector = new Connector();

$connector->connect('tcp://echo.example.com:7')
    ->then(function (React\Socket\ConnectionInterface $connection) {
        // Connection is a duplex stream
        
        // Read responses
        $connection->on('data', function (string $data) {
            echo "Received: {$data}";
        });
        
        // Write data
        $connection->write("Hello, server!\n");
        
        // Close after 5 seconds
        React\EventLoop\Loop::addTimer(5.0, function () use ($connection) {
            $connection->end();
        });
    });
```

## Stream Piping

Pipe data from one stream to another:

```php
<?php
require 'vendor/autoload.php';

use React\Stream\ReadableResourceStream;
use React\Stream\WritableResourceStream;
use React\Stream\ThroughStream;

// Source file
$source = new ReadableResourceStream(fopen('input.txt', 'r'));

// Transform stream (uppercase)
$transform = new ThroughStream(function (string $data) {
    return strtoupper($data);
});

// Destination file
$destination = new WritableResourceStream(fopen('output.txt', 'w'));

// Pipe: source -> transform -> destination
$source->pipe($transform)->pipe($destination);

$destination->on('close', function () {
    echo "File transformation complete\n";
});
```

## Through Streams for Transformation

```php
<?php
require 'vendor/autoload.php';

use React\Stream\ThroughStream;

// JSON line parser
$jsonParser = new ThroughStream(function (string $data) {
    $lines = explode("\n", $data);
    $objects = [];
    
    foreach ($lines as $line) {
        if (trim($line) !== '') {
            $objects[] = json_decode($line, true);
        }
    }
    
    return $objects;
});

// Compression stream
$compressor = new ThroughStream(function (string $data) {
    return gzcompress($data);
});

// Rate limiter (with state)
class RateLimitStream extends ThroughStream {
    private int $bytesThisSecond = 0;
    private int $maxBytesPerSecond;
    
    public function __construct(int $maxBytesPerSecond) {
        $this->maxBytesPerSecond = $maxBytesPerSecond;
        
        // Reset counter every second
        React\EventLoop\Loop::addPeriodicTimer(1.0, function () {
            $this->bytesThisSecond = 0;
        });
        
        parent::__construct(function (string $data) {
            $this->bytesThisSecond += strlen($data);
            
            if ($this->bytesThisSecond > $this->maxBytesPerSecond) {
                // Would exceed rate limit - buffer or drop
                return '';
            }
            
            return $data;
        });
    }
}
```

## Buffered Streams

```php
<?php
require 'vendor/autoload.php';

use React\Stream\CompositeStream;
use React\Stream\ThroughStream;

// Buffer until newline
class LineBuffer extends ThroughStream {
    private string $buffer = '';
    
    public function __construct() {
        parent::__construct(function (string $data) {
            $this->buffer .= $data;
            $lines = [];
            
            while (($pos = strpos($this->buffer, "\n")) !== false) {
                $lines[] = substr($this->buffer, 0, $pos);
                $this->buffer = substr($this->buffer, $pos + 1);
            }
            
            return implode("\n", $lines) . (count($lines) ? "\n" : '');
        });
    }
}

// Usage
$lineBuffer = new LineBuffer();

$lineBuffer->on('data', function (string $data) {
    // Each data event contains complete lines
    foreach (explode("\n", trim($data)) as $line) {
        if ($line !== '') {
            echo "Complete line: {$line}\n";
        }
    }
});

// Simulate chunked input
$lineBuffer->write("Hello, ");
$lineBuffer->write("World!\nThis is ");
$lineBuffer->write("line two\n");
```

## Back Pressure

Handle slow consumers:

```php
<?php
$readable->on('data', function (string $data) use ($writable, $readable) {
    // write() returns false if buffer is full
    $canContinue = $writable->write($data);
    
    if (!$canContinue) {
        // Pause reading until buffer drains
        $readable->pause();
    }
});

$writable->on('drain', function () use ($readable) {
    // Buffer drained, resume reading
    $readable->resume();
});

// Or simply use pipe() which handles this automatically
$readable->pipe($writable);
```

## Common Pitfalls
1. **Not setting streams to non-blocking mode** - Forgetting stream_set_blocking(false) causes fread() to block, defeating the purpose of async I/O.
2. **Confusing empty string with false from fread** - An empty string means no data yet; false means an error occurred. Both need different handling.
3. **Ignoring stream_select() modifying arrays** - stream_select() modifies the arrays passed to it, so you must copy them before each call.

## Best Practices
1. **Always set non-blocking mode for async I/O** - Call stream_set_blocking($stream, false) before using streams with an event loop.
2. **Use stream_select() for multiplexing** - Monitor multiple streams simultaneously instead of polling them sequentially.
3. **Handle all edge cases in stream reads** - Check for empty string (no data), false (error), and EOF condition separately.

## Summary
- stream_set_blocking(false) enables non-blocking I/O operations.
- stream_select() monitors multiple streams simultaneously for read/write readiness.
- stream_socket_client() with STREAM_CLIENT_ASYNC_CONNECT creates non-blocking connections.
- stream_socket_server() combined with stream_select() enables handling many concurrent clients.
- Always handle timeouts, EOF conditions, and errors when working with async streams.

## Resources

- [ReactPHP Stream Documentation](https://reactphp.org/stream/) — Complete guide to ReactPHP streams

---

> 📘 *This lesson is part of the [Asynchronous PHP](https://stanza.dev/courses/php-async) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*