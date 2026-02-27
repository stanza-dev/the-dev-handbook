---
source_course: "python-concurrency"
source_lesson: "python-concurrency-async-patterns"
---

# Common Async Patterns

## Introduction
Knowing async/await syntax is just the beginning. Real-world async code relies on recurring patterns for handling blocking calls, enforcing timeouts, limiting concurrency, and building producer-consumer pipelines. This lesson covers the patterns you will reach for most often when building production asyncio applications.

## Key Concepts
- **Blocking call**: A synchronous function that halts the entire thread (and therefore the event loop) until it returns.
- **run_in_executor()**: Offloads a blocking call to a thread pool so the event loop stays responsive.
- **Timeout**: A maximum wait duration after which an operation is cancelled.
- **Semaphore**: A concurrency primitive that limits how many tasks can enter a critical section simultaneously.
- **Async Queue**: A thread-safe, await-compatible queue for producer-consumer workflows.

## Real World Context
A web scraper needs to fetch 10,000 pages, but the target server rate-limits to 10 concurrent connections. An `asyncio.Semaphore(10)` ensures you never exceed that limit, while `asyncio.timeout()` ensures no single request blocks the entire pipeline. Some pages need CPU-heavy HTML parsing, so you offload that to an executor to keep the event loop free.

## Deep Dive

### Avoiding Blocking Calls

```python
import asyncio

# WRONG: Blocks the entire event loop
async def bad():
    import time
    time.sleep(5)  # Blocks everything!

# RIGHT: Non-blocking
async def good():
    await asyncio.sleep(5)  # Other tasks can run

# Running blocking code in executor
async def run_blocking():
    loop = asyncio.get_running_loop()
    result = await loop.run_in_executor(
        None,  # Default executor
        blocking_function,
        arg1, arg2
    )
```

### Timeouts

```python
async def main():
    try:
        async with asyncio.timeout(5.0):
            result = await slow_operation()
    except TimeoutError:
        print("Operation timed out")

# Or with wait_for
try:
    result = await asyncio.wait_for(slow_operation(), timeout=5.0)
except asyncio.TimeoutError:
    print("Timed out")
```

### Semaphores for Rate Limiting

```python
async def fetch_with_limit(url, semaphore):
    async with semaphore:  # Only N concurrent requests
        return await fetch(url)

async def main():
    semaphore = asyncio.Semaphore(10)  # Max 10 concurrent
    urls = [f"url{i}" for i in range(100)]
    
    tasks = [fetch_with_limit(url, semaphore) for url in urls]
    results = await asyncio.gather(*tasks)
```

### Async Queue

```python
async def producer(queue):
    for i in range(10):
        await queue.put(i)
        await asyncio.sleep(0.1)

async def consumer(queue):
    while True:
        item = await queue.get()
        print(f"Processing {item}")
        queue.task_done()

async def main():
    queue = asyncio.Queue()
    
    producers = [asyncio.create_task(producer(queue)) for _ in range(2)]
    consumers = [asyncio.create_task(consumer(queue)) for _ in range(3)]
    
    await asyncio.gather(*producers)
    await queue.join()  # Wait until all items processed
    
    for c in consumers:
        c.cancel()
```

## Common Pitfalls
1. **Using time.sleep() instead of asyncio.sleep()** — `time.sleep()` is a synchronous blocking call that freezes the entire event loop. All other tasks stop making progress. Always use `await asyncio.sleep()` for delays.
2. **Forgetting to cancel consumers** — In a producer-consumer pattern, consumers loop forever waiting for items. If you do not cancel them after the work is done, the program hangs.
3. **Setting semaphore value too high** — A semaphore of 1000 defeats the purpose of rate limiting. Match the value to the external constraint (server rate limit, database connection pool size, etc.).

## Best Practices
1. **Use `asyncio.timeout()` (Python 3.11+) instead of `wait_for()`** — The context manager form is cleaner, composes well with `async with`, and clearly scopes the timeout to a block of code.
2. **Offload blocking calls with `run_in_executor()`** — When you must call synchronous libraries (e.g., legacy database drivers), wrap them in an executor to keep the event loop responsive.

## Summary
- Never use synchronous blocking calls like `time.sleep()` inside async functions; use their async equivalents or `run_in_executor()`.
- Use `asyncio.timeout()` or `asyncio.wait_for()` to prevent operations from running indefinitely.
- Semaphores limit the number of concurrent tasks, ideal for rate-limited APIs.
- Async queues enable efficient producer-consumer workflows in asyncio.
- Always clean up consumer tasks when the work is complete.

## Code Examples

**Retry and timeout patterns**

```python
import asyncio

# Retry pattern
async def fetch_with_retry(url, max_retries=3):
    for attempt in range(max_retries):
        try:
            return await fetch(url)
        except Exception as e:
            if attempt == max_retries - 1:
                raise
            await asyncio.sleep(2 ** attempt)  # Exponential backoff

# Timeout with default value
async def fetch_with_default(url, timeout=5.0, default=None):
    try:
        async with asyncio.timeout(timeout):
            return await fetch(url)
    except TimeoutError:
        return default
```


## Resources

- [asyncio Timeout](https://docs.python.org/3.14/library/asyncio-task.html#asyncio.timeout) — Timeout context managers and patterns in asyncio

---

> 📘 *This lesson is part of the [Python Concurrency: Asyncio & Free-Threading](https://stanza.dev/courses/python-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*