---
source_course: "python-concurrency"
source_lesson: "python-concurrency-async-patterns"
---

# Async Programming Patterns

## Avoiding Blocking Calls

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
    loop = asyncio.get_event_loop()
    result = await loop.run_in_executor(
        None,  # Default executor
        blocking_function,
        arg1, arg2
    )
```

## Timeouts

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

## Semaphores for Rate Limiting

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

## Async Queue

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


---

> 📘 *This lesson is part of the [Python Concurrency: Asyncio & No-GIL](https://stanza.dev/courses/python-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*