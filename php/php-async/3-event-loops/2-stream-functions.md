---
source_course: "php-async"
source_lesson: "php-async-stream-functions"
---

# PHP Stream Functions for Async I/O

## Introduction
PHP's built-in stream functions are the foundation for non-blocking I/O. Before using higher-level libraries, understanding these low-level primitives helps you grasp how async I/O actually works under the hood and troubleshoot issues effectively.
## Key Concepts
- **stream_set_blocking()**: Controls whether stream operations block (wait for data) or return immediately. Setting false enables non-blocking mode.
- **stream_select()**: The core I/O multiplexing function that waits for multiple streams to become ready for reading, writing, or have errors.
- **stream_socket_client()**: Creates client connections with optional async mode (STREAM_CLIENT_ASYNC_CONNECT) for non-blocking connection establishment.
- **stream_socket_server()**: Creates a server socket that can accept multiple connections, the foundation for building async servers.
- **Stream Context**: Configuration options for streams including HTTP method, headers, SSL verification, and timeouts.
## Real World Context
PHP's stream functions power everything from file uploads to WebSocket servers. In production, non-blocking streams enable a single PHP process to handle thousands of simultaneous connections, which is essential for real-time features.

## Deep Dive
## Essential Stream Functions

### stream_set_blocking()

Controls whether operations on a stream block:

```php
<?php
// Blocking mode (default)
$fp = fopen('large_file.txt', 'r');
$data = fread($fp, 1000000);  // Waits until data is read

// Non-blocking mode
$fp = fopen('large_file.txt', 'r');
stream_set_blocking($fp, false);
$data = fread($fp, 1000000);  // Returns immediately
// $data may contain less than requested (or empty string)
```

### stream_select()

The heart of I/O multiplexing—waits for multiple streams to become ready:

```php
<?php
/**
 * stream_select(
 *     ?array &$read,    // Streams to check for reading
 *     ?array &$write,   // Streams to check for writing
 *     ?array &$except,  // Streams to check for exceptions
 *     ?int $seconds,    // Timeout seconds (null = block forever)
 *     int $microseconds // Timeout microseconds
 * ): int|false         // Number of ready streams
 */

// Example: Wait for multiple sockets
$sockets = [
    stream_socket_client('tcp://api1.example.com:80'),
    stream_socket_client('tcp://api2.example.com:80'),
    stream_socket_client('tcp://api3.example.com:80'),
];

foreach ($sockets as $socket) {
    stream_set_blocking($socket, false);
    fwrite($socket, "GET / HTTP/1.0\r\nHost: example.com\r\n\r\n");
}

// Poll until all responses received
$responses = array_fill(0, count($sockets), '');
$active = $sockets;

while (!empty($active)) {
    $read = $active;  // Copy because stream_select modifies the array
    $write = null;
    $except = null;
    
    $ready = stream_select($read, $write, $except, 1);  // 1 second timeout
    
    if ($ready === false) {
        throw new RuntimeException('stream_select failed');
    }
    
    if ($ready === 0) {
        echo "Timeout, checking again...\n";
        continue;
    }
    
    foreach ($read as $socket) {
        $index = array_search($socket, $sockets, true);
        $chunk = fread($socket, 8192);
        
        if ($chunk === '' || $chunk === false) {
            // Socket closed
            fclose($socket);
            unset($active[array_search($socket, $active, true)]);
            echo "Socket {$index} complete: " . strlen($responses[$index]) . " bytes\n";
        } else {
            $responses[$index] .= $chunk;
        }
    }
}

echo "All responses received!\n";
```

### stream_socket_client()

Creates client connections with async support:

```php
<?php
// Synchronous connection
$sync = stream_socket_client('tcp://example.com:80');

// Asynchronous connection (returns immediately)
$async = stream_socket_client(
    'tcp://example.com:80',
    $errno,
    $errstr,
    30,  // Timeout for DNS resolution
    STREAM_CLIENT_CONNECT | STREAM_CLIENT_ASYNC_CONNECT
);

// Check if connection is ready
stream_set_blocking($async, false);
$write = [$async];
$read = null;
$except = null;

if (stream_select($read, $write, $except, 5) > 0) {
    // Connection established, can send data
    fwrite($async, "GET / HTTP/1.0\r\n\r\n");
}
```

### stream_socket_server()

Creates servers that can accept multiple connections:

```php
<?php
$server = stream_socket_server(
    'tcp://0.0.0.0:8080',
    $errno,
    $errstr,
    STREAM_SERVER_BIND | STREAM_SERVER_LISTEN
);

if ($server === false) {
    die("Failed to create server: $errstr");
}

stream_set_blocking($server, false);

$clients = [];

while (true) {
    // Check for new connections and readable clients
    $read = array_merge([$server], $clients);
    $write = null;
    $except = null;
    
    if (stream_select($read, $write, $except, 1) > 0) {
        // New connection?
        if (in_array($server, $read, true)) {
            $client = stream_socket_accept($server, 0);
            if ($client) {
                stream_set_blocking($client, false);
                $clients[(int)$client] = $client;
                echo "New client connected\n";
            }
        }
        
        // Data from existing clients?
        foreach ($read as $stream) {
            if ($stream === $server) continue;
            
            $data = fread($stream, 1024);
            if ($data === '' || $data === false) {
                // Client disconnected
                unset($clients[(int)$stream]);
                fclose($stream);
                echo "Client disconnected\n";
            } else {
                // Echo back
                fwrite($stream, "Echo: {$data}");
            }
        }
    }
}
```

## Stream Contexts

```php
<?php
// Create context with options
$context = stream_context_create([
    'http' => [
        'method' => 'POST',
        'header' => "Content-Type: application/json\r\n",
        'content' => json_encode(['key' => 'value']),
        'timeout' => 10.0,
    ],
    'ssl' => [
        'verify_peer' => true,
        'verify_peer_name' => true,
    ],
]);

// Use with file functions
$response = file_get_contents('https://api.example.com/data', false, $context);

// Or with streams
$stream = fopen('https://api.example.com/data', 'r', false, $context);
```

## Useful Stream Metadata

```php
<?php
$stream = fopen('http://example.com', 'r');

// Get metadata about the stream
$meta = stream_get_meta_data($stream);

print_r($meta);
/*
Array (
    [timed_out] => false
    [blocked] => true
    [eof] => false
    [wrapper_type] => http
    [stream_type] => tcp_socket/ssl
    [mode] => r
    [unread_bytes] => 0
    [seekable] => false
    [uri] => http://example.com
)
*/

// Check for timeout
if ($meta['timed_out']) {
    echo "Connection timed out!\n";
}

// Check if at end of file
if ($meta['eof']) {
    echo "Reached end of stream\n";
}
```

## Common Patterns

### Timeout Wrapper

```php
<?php
function readWithTimeout($stream, int $bytes, float $timeout): string|false {
    $start = microtime(true);
    $data = '';
    
    stream_set_blocking($stream, false);
    
    while (strlen($data) < $bytes) {
        $elapsed = microtime(true) - $start;
        if ($elapsed >= $timeout) {
            return strlen($data) > 0 ? $data : false;
        }
        
        $remaining = $timeout - $elapsed;
        $seconds = (int) $remaining;
        $microseconds = (int) (($remaining - $seconds) * 1000000);
        
        $read = [$stream];
        $write = null;
        $except = null;
        
        if (stream_select($read, $write, $except, $seconds, $microseconds) > 0) {
            $chunk = fread($stream, $bytes - strlen($data));
            if ($chunk === '' || $chunk === false) {
                break;  // EOF or error
            }
            $data .= $chunk;
        }
    }
    
    return $data;
}
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

- [PHP Manual - Streams](https://www.php.net/manual/en/book.stream.php) — Complete PHP stream functions reference

---

> 📘 *This lesson is part of the [Asynchronous PHP](https://stanza.dev/courses/php-async) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*