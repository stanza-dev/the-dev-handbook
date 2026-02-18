---
source_course: "php-async"
source_lesson: "php-async-event-loop-basics"
---

# Understanding Event Loops

The **event loop** is the heart of asynchronous programming. It's a programming construct that waits for and dispatches events or messages in a program.

## What is an Event Loop?

An event loop continuously:
1. Checks for pending events (I/O ready, timers expired, etc.)
2. Dispatches callbacks for ready events
3. Repeats until no more work remains

```php
<?php
// Conceptual event loop structure
function eventLoop(): void {
    while ($hasWorkRemaining) {
        // 1. Wait for events (with timeout)
        $events = waitForEvents($timeout);
        
        // 2. Process timers
        foreach ($expiredTimers as $timer) {
            $timer->callback();
        }
        
        // 3. Process I/O events
        foreach ($events as $event) {
            $event->handler();
        }
        
        // 4. Process immediate callbacks
        while ($immediateCallback = $immediateQueue->dequeue()) {
            $immediateCallback();
        }
    }
}
```

## Event Loop Phases

```
   ┌───────────────────────────┐
┌─▶│         Timers            │
│  │  (setTimeout, setInterval)│
│  └─────────────┬─────────────┘
│                │
│  ┌─────────────▼─────────────┐
│  │     Pending Callbacks     │
│  │  (I/O callbacks deferred) │
│  └─────────────┬─────────────┘
│                │
│  ┌─────────────▼─────────────┐
│  │       Poll (I/O)          │
│  │ (retrieve new I/O events) │
│  └─────────────┬─────────────┘
│                │
│  ┌─────────────▼─────────────┐
│  │         Check             │
│  │   (setImmediate callbacks)│
│  └─────────────┬─────────────┘
│                │
│  ┌─────────────▼─────────────┐
│  │     Close Callbacks       │
│  │  (cleanup, socket close)  │
│  └─────────────┬─────────────┘
│                │
└────────────────┘
```

## Building a Simple Event Loop

```php
<?php
class EventLoop {
    /** @var array<int, array{callback: callable, time: float}> */
    private array $timers = [];
    
    /** @var SplQueue<callable> */
    private SplQueue $deferred;
    
    /** @var array<int, array{stream: resource, callback: callable}> */
    private array $readers = [];
    
    private bool $running = false;
    private int $timerId = 0;
    
    public function __construct() {
        $this->deferred = new SplQueue();
    }
    
    public function addTimer(float $seconds, callable $callback): int {
        $id = ++$this->timerId;
        $this->timers[$id] = [
            'callback' => $callback,
            'time' => microtime(true) + $seconds
        ];
        return $id;
    }
    
    public function cancelTimer(int $id): void {
        unset($this->timers[$id]);
    }
    
    public function defer(callable $callback): void {
        $this->deferred->enqueue($callback);
    }
    
    public function addReader($stream, callable $callback): void {
        $id = (int) $stream;
        $this->readers[$id] = [
            'stream' => $stream,
            'callback' => $callback
        ];
    }
    
    public function removeReader($stream): void {
        $id = (int) $stream;
        unset($this->readers[$id]);
    }
    
    public function run(): void {
        $this->running = true;
        
        while ($this->running && $this->hasWork()) {
            $this->tick();
        }
    }
    
    public function stop(): void {
        $this->running = false;
    }
    
    private function hasWork(): bool {
        return !empty($this->timers) 
            || !$this->deferred->isEmpty() 
            || !empty($this->readers);
    }
    
    private function tick(): void {
        // Process deferred callbacks
        $count = $this->deferred->count();
        for ($i = 0; $i < $count; $i++) {
            $callback = $this->deferred->dequeue();
            $callback();
        }
        
        // Process timers
        $now = microtime(true);
        foreach ($this->timers as $id => $timer) {
            if ($timer['time'] <= $now) {
                unset($this->timers[$id]);
                $timer['callback']();
            }
        }
        
        // Check for readable streams
        if (!empty($this->readers)) {
            $this->pollStreams();
        } else {
            // No I/O, small sleep to prevent CPU spinning
            usleep(1000);
        }
    }
    
    private function pollStreams(): void {
        $read = array_column($this->readers, 'stream');
        $write = null;
        $except = null;
        
        // Wait up to 10ms for activity
        if (stream_select($read, $write, $except, 0, 10000) > 0) {
            foreach ($read as $stream) {
                $id = (int) $stream;
                if (isset($this->readers[$id])) {
                    $this->readers[$id]['callback']($stream);
                }
            }
        }
    }
}
```

## Using the Event Loop

```php
<?php
$loop = new EventLoop();

// Add a timer
$loop->addTimer(2.0, function() {
    echo "2 seconds elapsed!\n";
});

// Add periodic timer (recreates itself)
$periodicCallback = function() use ($loop, &$periodicCallback) {
    echo "Tick at " . date('H:i:s') . "\n";
    $loop->addTimer(1.0, $periodicCallback);
};
$loop->addTimer(1.0, $periodicCallback);

// Stop after 5 seconds
$loop->addTimer(5.0, function() use ($loop) {
    echo "Stopping...\n";
    $loop->stop();
});

// Start the loop
$loop->run();
echo "Event loop finished\n";
```

## Non-Blocking I/O with Streams

```php
<?php
// Non-blocking HTTP request
$loop = new EventLoop();

$socket = stream_socket_client(
    'tcp://httpbin.org:80',
    $errno,
    $errstr,
    30,
    STREAM_CLIENT_CONNECT | STREAM_CLIENT_ASYNC_CONNECT
);

if ($socket === false) {
    die("Failed to connect: $errstr");
}

// Set non-blocking mode
stream_set_blocking($socket, false);

// Send request when connected
$request = "GET /get HTTP/1.1\r\nHost: httpbin.org\r\nConnection: close\r\n\r\n";
fwrite($socket, $request);

// Read response asynchronously
$response = '';
$loop->addReader($socket, function($stream) use ($loop, &$response) {
    $chunk = fread($stream, 8192);
    
    if ($chunk === '' || $chunk === false) {
        // Connection closed
        $loop->removeReader($stream);
        fclose($stream);
        echo "Response received: " . strlen($response) . " bytes\n";
        $loop->stop();
    } else {
        $response .= $chunk;
    }
});

// Timeout
$loop->addTimer(10.0, function() use ($loop) {
    echo "Timeout!\n";
    $loop->stop();
});

$loop->run();
```

## Key Concepts

| Concept | Description |
|---------|-------------|
| **Non-blocking** | Operations return immediately, even if incomplete |
| **Callback** | Function called when an event occurs |
| **Polling** | Checking if events are ready (`stream_select`) |
| **Event dispatch** | Running callbacks for ready events |
| **Tick** | One iteration of the event loop |

## Resources

- [PHP Manual - stream_select](https://www.php.net/manual/en/function.stream-select.php) — Core function for non-blocking I/O multiplexing

---

> 📘 *This lesson is part of the [Asynchronous PHP](https://stanza.dev/courses/php-async) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*