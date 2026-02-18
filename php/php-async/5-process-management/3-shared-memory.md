---
source_course: "php-async"
source_lesson: "php-async-ipc-shared-memory"
---

# Inter-Process Communication (IPC)

When using multiple processes, they need ways to share data. PHP offers several IPC mechanisms.

## IPC Methods Overview

| Method | Speed | Complexity | Use Case |
|--------|-------|------------|----------|
| Files | Slow | Simple | Persistent data, logs |
| Pipes | Fast | Medium | Parent-child communication |
| Shared Memory | Very Fast | Complex | High-performance data sharing |
| Message Queues | Medium | Medium | Task distribution |
| Sockets | Medium | Medium | Network-capable IPC |

## Using Pipes

```php
<?php
// Create a pipe
$descriptors = [
    0 => ['pipe', 'r'],  // stdin
    1 => ['pipe', 'w'],  // stdout
    2 => ['pipe', 'w'],  // stderr
];

$process = proc_open('php worker.php', $descriptors, $pipes);

if ($process) {
    // Send data to child's stdin
    fwrite($pipes[0], json_encode(['task' => 'process', 'data' => [1,2,3]]));
    fclose($pipes[0]);
    
    // Read from child's stdout
    $output = stream_get_contents($pipes[1]);
    fclose($pipes[1]);
    
    // Read errors
    $errors = stream_get_contents($pipes[2]);
    fclose($pipes[2]);
    
    $returnCode = proc_close($process);
    
    echo "Output: {$output}\n";
    echo "Return code: {$returnCode}\n";
}
```

## Manual Pipe Creation

```php
<?php
// Create pipe pair
$sockets = [];
if (!socket_create_pair(AF_UNIX, SOCK_STREAM, 0, $sockets)) {
    die("Could not create socket pair\n");
}

$pid = pcntl_fork();

if ($pid === 0) {
    // Child: close parent's end
    socket_close($sockets[0]);
    
    // Read message from parent
    $msg = socket_read($sockets[1], 1024);
    echo "Child received: {$msg}\n";
    
    // Send response
    socket_write($sockets[1], "Hello from child!");
    
    socket_close($sockets[1]);
    exit(0);
} else {
    // Parent: close child's end
    socket_close($sockets[1]);
    
    // Send message to child
    socket_write($sockets[0], "Hello from parent!");
    
    // Read response
    $response = socket_read($sockets[0], 1024);
    echo "Parent received: {$response}\n";
    
    socket_close($sockets[0]);
    pcntl_waitpid($pid, $status);
}
```

## Shared Memory (shmop)

```php
<?php
// Create shared memory segment
$key = ftok(__FILE__, 'a');  // Generate unique key
$size = 1024;  // bytes

// Parent creates shared memory
$shmId = shmop_open($key, 'c', 0644, $size);

if ($shmId === false) {
    die("Could not create shared memory\n");
}

$pid = pcntl_fork();

if ($pid === 0) {
    // Child: open existing segment
    $shmId = shmop_open($key, 'w', 0, 0);
    
    // Write data
    $data = json_encode(['result' => 42, 'time' => time()]);
    shmop_write($shmId, $data, 0);
    
    echo "Child wrote to shared memory\n";
    exit(0);
} else {
    // Wait for child
    pcntl_waitpid($pid, $status);
    
    // Read data
    $data = shmop_read($shmId, 0, $size);
    $data = trim($data, "\0");  // Remove null padding
    
    echo "Parent read: {$data}\n";
    
    // Cleanup
    shmop_delete($shmId);
}
```

## System V Shared Memory

```php
<?php
// More features than shmop
$key = ftok(__FILE__, 'b');

// Create shared memory segment
$shmId = shm_attach($key, 1024, 0644);

if ($shmId === false) {
    die("Could not attach shared memory\n");
}

$pid = pcntl_fork();

if ($pid === 0) {
    // Child writes
    $shmId = shm_attach($key);
    
    shm_put_var($shmId, 1, ['count' => 100]);
    shm_put_var($shmId, 2, 'hello');
    
    exit(0);
} else {
    pcntl_waitpid($pid, $status);
    
    // Parent reads
    if (shm_has_var($shmId, 1)) {
        $data = shm_get_var($shmId, 1);
        print_r($data);  // ['count' => 100]
    }
    
    // Cleanup
    shm_remove($shmId);
}
```

## Message Queues

```php
<?php
$key = ftok(__FILE__, 'q');
$queue = msg_get_queue($key, 0644);

if ($queue === false) {
    die("Could not create message queue\n");
}

$pid = pcntl_fork();

if ($pid === 0) {
    // Child: send messages
    for ($i = 0; $i < 5; $i++) {
        $message = ['task_id' => $i, 'data' => "Task {$i}"];
        msg_send($queue, 1, $message);
        echo "Sent task {$i}\n";
    }
    exit(0);
} else {
    pcntl_waitpid($pid, $status);
    
    // Parent: receive messages
    while (msg_stat_queue($queue)['msg_qnum'] > 0) {
        $msgType = null;
        $message = null;
        
        if (msg_receive($queue, 0, $msgType, 1024, $message, true, MSG_IPC_NOWAIT)) {
            echo "Received: " . json_encode($message) . "\n";
        }
    }
    
    // Cleanup
    msg_remove_queue($queue);
}
```

## Worker Pool with IPC

```php
<?php
class WorkerPool {
    private int $workerCount;
    private array $workers = [];
    private $queue;
    private $resultMemory;
    
    public function __construct(int $workerCount = 4) {
        $this->workerCount = $workerCount;
        $this->queue = msg_get_queue(ftok(__FILE__, 'q'));
        $this->resultMemory = shm_attach(ftok(__FILE__, 's'), 65536);
    }
    
    public function start(): void {
        for ($i = 0; $i < $this->workerCount; $i++) {
            $pid = pcntl_fork();
            
            if ($pid === 0) {
                $this->runWorker($i);
                exit(0);
            }
            
            $this->workers[$pid] = $i;
        }
    }
    
    private function runWorker(int $workerId): void {
        while (true) {
            $msgType = null;
            $task = null;
            
            // Block waiting for task (type 1 = tasks)
            if (msg_receive($this->queue, 1, $msgType, 1024, $task, true)) {
                if ($task === 'STOP') {
                    break;
                }
                
                // Process task
                $result = $this->processTask($task);
                
                // Store result in shared memory
                $results = shm_has_var($this->resultMemory, 1) 
                    ? shm_get_var($this->resultMemory, 1) 
                    : [];
                $results[$task['id']] = $result;
                shm_put_var($this->resultMemory, 1, $results);
            }
        }
    }
    
    private function processTask(array $task): mixed {
        // Simulate processing
        usleep($task['duration'] * 1000);
        return ['id' => $task['id'], 'result' => $task['value'] * 2];
    }
    
    public function submit(array $task): void {
        msg_send($this->queue, 1, $task);
    }
    
    public function shutdown(): array {
        // Send stop signals
        for ($i = 0; $i < $this->workerCount; $i++) {
            msg_send($this->queue, 1, 'STOP');
        }
        
        // Wait for all workers
        foreach ($this->workers as $pid => $workerId) {
            pcntl_waitpid($pid, $status);
        }
        
        // Get results
        $results = shm_get_var($this->resultMemory, 1);
        
        // Cleanup
        shm_remove($this->resultMemory);
        msg_remove_queue($this->queue);
        
        return $results;
    }
}

// Usage
$pool = new WorkerPool(4);
$pool->start();

// Submit tasks
for ($i = 0; $i < 20; $i++) {
    $pool->submit([
        'id' => $i,
        'value' => $i * 10,
        'duration' => rand(100, 500)
    ]);
}

$results = $pool->shutdown();
print_r($results);
```

## Resources

- [PHP Manual - Shared Memory](https://www.php.net/manual/en/book.shmop.php) — PHP shared memory functions documentation

---

> 📘 *This lesson is part of the [Asynchronous PHP](https://stanza.dev/courses/php-async) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*