---
source_course: "php-async"
source_lesson: "php-async-blocking-operations"
---

# Understanding Blocking Operations

Before diving into asynchronous programming, you need to understand what makes PHP traditionally synchronous and why blocking operations can be problematic for certain applications.

## What is Synchronous Execution?

**Synchronous execution** means your code runs line by line, in order, waiting for each operation to complete before moving to the next one. This is how PHP has traditionally worked and how most developers first learn to program.

```php
<?php
// Synchronous execution example
echo "Starting...\n";

// This blocks - nothing else happens until it completes
$data = file_get_contents('https://api.example.com/users');

echo "Got data!\n";  // Only runs after file_get_contents finishes

// Process the data
$users = json_decode($data, true);
echo "Processed " . count($users) . " users\n";
```

In this example, the entire script waits while `file_get_contents()` fetches data from the API. If the API takes 3 seconds to respond, your script does **absolutely nothing** for those 3 seconds.

## What is a Blocking Operation?

A **blocking operation** is any operation that causes your program to pause and wait. Common blocking operations in PHP include:

| Operation Type | Examples |
|----------------|----------|
| **Network I/O** | HTTP requests, database queries, API calls |
| **File I/O** | Reading/writing files, especially large ones |
| **Sleep functions** | `sleep()`, `usleep()` |
| **External processes** | `shell_exec()`, `exec()` |
| **User input** | `fgets(STDIN)` in CLI scripts |

## The Problem with Blocking

Imagine you need to fetch data from three different APIs:

```php
<?php
// Sequential blocking - SLOW!
$startTime = microtime(true);

// Each call blocks until complete
$users = file_get_contents('https://api.example.com/users');     // 2 seconds
$orders = file_get_contents('https://api.example.com/orders');   // 1 second
$products = file_get_contents('https://api.example.com/products'); // 1.5 seconds

$totalTime = microtime(true) - $startTime;
echo "Total time: {$totalTime} seconds\n"; // ~4.5 seconds
```

Even though these three requests are **independent** of each other, they run sequentially. The total time is the **sum** of all individual times.

## Visualizing the Timeline

```
Synchronous Execution:

|-- Users API (2s) --|-- Orders API (1s) --|-- Products API (1.5s) --|
                                                                      Total: 4.5s

Asynchronous Execution (ideal):

|-- Users API (2s) --|
|-- Orders API (1s) --|
|-- Products API (1.5s) --|
                          Total: 2s (longest single request)
```

With asynchronous execution, all three requests could run **concurrently**, and the total time would only be as long as the slowest request.

## When Blocking is Acceptable

Blocking isn't always bad. For many PHP applications, synchronous execution is perfectly fine:

1. **Simple web requests**: Most web pages complete in under 200ms
2. **CRUD operations**: Basic database operations are fast
3. **Sequential dependencies**: When step B requires step A's result
4. **Low traffic applications**: Blocking matters less with few users

## When Async Becomes Necessary

Consider asynchronous programming when:

- **High concurrency**: Many simultaneous users or requests
- **External API dependencies**: Multiple third-party services
- **Long-running tasks**: File processing, report generation
- **Real-time features**: Chat, notifications, live updates
- **WebSocket servers**: Persistent connections

## CPU-Bound vs I/O-Bound

Understanding this distinction is crucial:

**I/O-Bound Operations** (good candidates for async):
- Network requests
- Database queries  
- File reading/writing
- External service calls

These operations spend most time **waiting** for external systems.

**CPU-Bound Operations** (async doesn't help much):
- Complex calculations
- Image processing
- Data encryption
- Sorting large datasets

These operations keep the CPU busy. Async won't speed them up because there's nothing to wait for—the CPU is already working.

## PHP's Traditional Model

PHP was designed for the **request-response cycle**:

1. Web server receives HTTP request
2. PHP script starts
3. Script executes synchronously
4. Response sent to client
5. Script terminates, memory freed

This model is simple and effective for most web applications. Each request is isolated, making PHP naturally scalable through multiple processes.

## Resources

- [PHP Manual - Introduction](https://www.php.net/manual/en/intro-whatis.php) — Official overview of what PHP is and its execution model

---

> 📘 *This lesson is part of the [Asynchronous PHP](https://stanza.dev/courses/php-async) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*