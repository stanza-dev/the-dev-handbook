---
source_course: "php-async"
source_lesson: "php-async-signal-handling"
---

# Signal Handling in PHP

Signals are software interrupts sent to a process. Handling them properly is essential for robust CLI applications.

## Common Signals

| Signal | Number | Description | Default Action |
|--------|--------|-------------|----------------|
| SIGTERM | 15 | Termination request | Terminate |
| SIGINT | 2 | Interrupt (Ctrl+C) | Terminate |
| SIGKILL | 9 | Force kill | Cannot be caught |
| SIGHUP | 1 | Hangup | Terminate |
| SIGCHLD | 17 | Child status changed | Ignore |
| SIGUSR1 | 10 | User-defined 1 | Terminate |
| SIGUSR2 | 12 | User-defined 2 | Terminate |

## Registering Signal Handlers

```php
<?php
// Enable async signal handling
pcntl_async_signals(true);

// Register handler for SIGTERM
pcntl_signal(SIGTERM, function (int $signal) {
    echo "Received SIGTERM, shutting down gracefully...\n";
    // Cleanup code here
    exit(0);
});

// Register handler for SIGINT (Ctrl+C)
pcntl_signal(SIGINT, function (int $signal) {
    echo "\nReceived SIGINT (Ctrl+C)\n";
    exit(0);
});

// Multiple signals, same handler
$shutdown = function (int $signal) {
    $signalNames = [
        SIGTERM => 'SIGTERM',
        SIGINT => 'SIGINT',
        SIGHUP => 'SIGHUP',
    ];
    echo "Received " . ($signalNames[$signal] ?? $signal) . "\n";
    exit(0);
};

pcntl_signal(SIGTERM, $shutdown);
pcntl_signal(SIGINT, $shutdown);
pcntl_signal(SIGHUP, $shutdown);

// Main loop
echo "Running... Press Ctrl+C to stop\n";
while (true) {
    echo ".";
    sleep(1);
}
```

## Synchronous vs Async Signal Handling

```php
<?php
// METHOD 1: Async signals (PHP 7.1+, recommended)
pcntl_async_signals(true);
pcntl_signal(SIGTERM, $handler);
// Handler runs immediately when signal received

// METHOD 2: Manual dispatch (older method)
pcntl_signal(SIGTERM, $handler);
while (true) {
    // Must call this to process pending signals
    pcntl_signal_dispatch();
    
    // Your work here
    doWork();
}
```

## Graceful Shutdown Pattern

```php
<?php
class GracefulWorker {
    private bool $shouldStop = false;
    private array $activeJobs = [];
    
    public function __construct() {
        pcntl_async_signals(true);
        
        pcntl_signal(SIGTERM, [$this, 'handleShutdown']);
        pcntl_signal(SIGINT, [$this, 'handleShutdown']);
    }
    
    public function handleShutdown(int $signal): void {
        echo "\nShutdown requested, finishing current jobs...\n";
        $this->shouldStop = true;
    }
    
    public function run(): void {
        echo "Worker started. PID: " . getmypid() . "\n";
        
        while (!$this->shouldStop) {
            $job = $this->getNextJob();
            
            if ($job === null) {
                sleep(1);
                continue;
            }
            
            $this->activeJobs[] = $job;
            $this->processJob($job);
            array_pop($this->activeJobs);
        }
        
        // Wait for active jobs to complete
        while (!empty($this->activeJobs)) {
            echo "Waiting for " . count($this->activeJobs) . " jobs...\n";
            sleep(1);
        }
        
        echo "Worker stopped gracefully\n";
    }
    
    private function getNextJob(): ?array {
        // Simulate job queue
        return rand(0, 2) > 0 ? ['id' => uniqid()] : null;
    }
    
    private function processJob(array $job): void {
        echo "Processing job {$job['id']}\n";
        sleep(2);  // Simulate work
        echo "Completed job {$job['id']}\n";
    }
}

$worker = new GracefulWorker();
$worker->run();
```

## Child Process Signals

```php
<?php
pcntl_async_signals(true);

$children = [];

// Handle SIGCHLD to know when children exit
pcntl_signal(SIGCHLD, function () use (&$children) {
    // Reap all finished children
    while (($pid = pcntl_waitpid(-1, $status, WNOHANG)) > 0) {
        if (isset($children[$pid])) {
            $exitCode = pcntl_wexitstatus($status);
            echo "Child {$pid} exited with code {$exitCode}\n";
            unset($children[$pid]);
        }
    }
});

// Fork children
for ($i = 0; $i < 3; $i++) {
    $pid = pcntl_fork();
    
    if ($pid === 0) {
        sleep(rand(1, 5));
        exit($i);
    }
    
    $children[$pid] = true;
}

// Parent continues working
echo "Parent doing other work...\n";
while (!empty($children)) {
    echo "Working... (" . count($children) . " children)\n";
    sleep(1);
}
echo "All children finished\n";
```

## Sending Signals

```php
<?php
// Send signal to specific process
posix_kill($pid, SIGTERM);

// Send signal to process group
posix_kill(-$pgid, SIGTERM);  // Negative PID = process group

// From command line
// kill -TERM 1234
// kill -15 1234
// kill -SIGTERM 1234

// Check if process exists
if (posix_kill($pid, 0)) {
    echo "Process {$pid} is running\n";
} else {
    echo "Process {$pid} not found\n";
}
```

## Signal-Safe Operations

Signal handlers should be minimal. These operations are generally safe:

✅ Setting flags/variables
✅ Calling `exit()`
✅ Simple arithmetic

These should be avoided in handlers:

❌ Complex I/O operations
❌ Memory allocation
❌ Database operations
❌ File operations

```php
<?php
// Good pattern: set flag, handle in main loop
$shutdown = false;

pcntl_signal(SIGTERM, function () use (&$shutdown) {
    $shutdown = true;  // Just set flag
});

while (!$shutdown) {
    // Main loop handles the actual shutdown
    doWork();
    
    if ($shutdown) {
        performCleanup();  // Safe to do complex operations here
    }
}
```

## Resources

- [PHP Manual - pcntl_signal](https://www.php.net/manual/en/function.pcntl-signal.php) — Official documentation for signal handling

---

> 📘 *This lesson is part of the [Asynchronous PHP](https://stanza.dev/courses/php-async) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*