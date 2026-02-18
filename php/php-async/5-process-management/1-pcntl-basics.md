---
source_course: "php-async"
source_lesson: "php-async-pcntl-basics"
---

# Introduction to PCNTL

The PCNTL (Process Control) extension allows PHP to create and manage child processes, enabling true parallelism for CPU-bound tasks.

## What is PCNTL?

PCNTL provides Unix-style process control:

- **Fork processes**: Create child processes
- **Signal handling**: Respond to system signals
- **Process management**: Wait for children, get status

> ⚠️ PCNTL only works on Unix-like systems (Linux, macOS) and CLI mode. It doesn't work on Windows or in web server contexts.

## Checking Availability

```php
<?php
if (!extension_loaded('pcntl')) {
    die("PCNTL extension not available\n");
}

if (php_sapi_name() !== 'cli') {
    die("PCNTL only works in CLI mode\n");
}

echo "PCNTL is available!\n";
```

## Basic Forking

```php
<?php
$pid = pcntl_fork();

if ($pid === -1) {
    // Fork failed
    die("Could not fork process\n");
    
} elseif ($pid === 0) {
    // Child process
    echo "Child process (PID: " . getmypid() . ")\n";
    sleep(2);
    echo "Child done\n";
    exit(0);  // Important: child must exit!
    
} else {
    // Parent process
    echo "Parent process (PID: " . getmypid() . ")\n";
    echo "Created child with PID: {$pid}\n";
    
    // Wait for child to complete
    pcntl_waitpid($pid, $status);
    echo "Child exited with status: " . pcntl_wexitstatus($status) . "\n";
}

echo "Process " . getmypid() . " finishing\n";
```

Output:
```
Parent process (PID: 1234)
Created child with PID: 1235
Child process (PID: 1235)
Child done
Child exited with status: 0
Process 1234 finishing
```

## Understanding Fork

When you call `pcntl_fork()`:

1. The entire process is duplicated
2. Both processes continue from the same point
3. `fork()` returns differently in each process:
   - Parent: receives child's PID
   - Child: receives 0

```
Before fork:
┌─────────────────────┐
│ Process (PID 1234)  │
│ $x = 5              │
│ $y = 10             │
└─────────────────────┘

After fork:
┌─────────────────────┐     ┌─────────────────────┐
│ Parent (PID 1234)   │     │ Child (PID 1235)    │
│ $x = 5              │     │ $x = 5              │
│ $y = 10             │     │ $y = 10             │
│ $pid = 1235         │     │ $pid = 0            │
└─────────────────────┘     └─────────────────────┘
```

## Multiple Child Processes

```php
<?php
$workerCount = 4;
$children = [];

for ($i = 0; $i < $workerCount; $i++) {
    $pid = pcntl_fork();
    
    if ($pid === -1) {
        die("Fork failed\n");
    } elseif ($pid === 0) {
        // Child
        $workerId = $i;
        echo "Worker {$workerId} started (PID: " . getmypid() . ")\n";
        
        // Simulate work
        $result = doWork($workerId);
        
        echo "Worker {$workerId} finished with result: {$result}\n";
        exit($result);  // Exit code is limited to 0-255
    } else {
        // Parent
        $children[$pid] = $i;
    }
}

// Parent: wait for all children
echo "\nParent waiting for " . count($children) . " workers...\n\n";

while (count($children) > 0) {
    $pid = pcntl_waitpid(-1, $status);
    
    if ($pid > 0) {
        $workerId = $children[$pid];
        $exitCode = pcntl_wexitstatus($status);
        echo "Worker {$workerId} (PID {$pid}) exited with code {$exitCode}\n";
        unset($children[$pid]);
    }
}

echo "\nAll workers completed\n";

function doWork(int $workerId): int {
    // Simulate varying work times
    $sleepTime = rand(1, 3);
    sleep($sleepTime);
    return $sleepTime;  // Return as exit code
}
```

## Process Status Checking

```php
<?php
function waitForChild(int $pid): array {
    pcntl_waitpid($pid, $status);
    
    return [
        'exited_normally' => pcntl_wifexited($status),
        'exit_code' => pcntl_wexitstatus($status),
        'signaled' => pcntl_wifsignaled($status),
        'signal' => pcntl_wtermsig($status),
        'stopped' => pcntl_wifstopped($status),
    ];
}

$pid = pcntl_fork();

if ($pid === 0) {
    // Child - exit with specific code
    exit(42);
}

$info = waitForChild($pid);
print_r($info);
// Array ( [exited_normally] => 1 [exit_code] => 42 [signaled] => 0 ... )
```

## Non-Blocking Wait

```php
<?php
$children = [];

// Fork multiple children
for ($i = 0; $i < 3; $i++) {
    $pid = pcntl_fork();
    if ($pid === 0) {
        sleep(rand(1, 5));
        exit(0);
    }
    $children[] = $pid;
}

// Non-blocking poll for completed children
while (!empty($children)) {
    foreach ($children as $key => $pid) {
        // WNOHANG makes it non-blocking
        $result = pcntl_waitpid($pid, $status, WNOHANG);
        
        if ($result === $pid) {
            // Child completed
            echo "Child {$pid} completed\n";
            unset($children[$key]);
        } elseif ($result === -1) {
            // Error
            unset($children[$key]);
        }
        // result === 0 means still running
    }
    
    echo "Checking... " . count($children) . " still running\n";
    usleep(500000);  // 0.5 seconds
}
```

## Resources

- [PHP Manual - PCNTL Functions](https://www.php.net/manual/en/ref.pcntl.php) — Complete reference for PCNTL functions

---

> 📘 *This lesson is part of the [Asynchronous PHP](https://stanza.dev/courses/php-async) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*