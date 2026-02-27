---
source_course: "python-concurrency"
source_lesson: "python-concurrency-taskgroup-patterns"
---

# TaskGroup Patterns

## Introduction
TaskGroup is a versatile building block for structuring concurrent work. Beyond basic fan-out, it can be combined with batching, timeouts, semaphores, and queues to solve real production problems. This lesson presents four essential TaskGroup patterns that you will use repeatedly in async Python applications.

## Key Concepts
- **Batching**: Processing items in fixed-size groups to control memory usage and concurrency.
- **Timeout wrapping**: Nesting a TaskGroup inside `asyncio.timeout()` to enforce a deadline on all tasks.
- **Semaphore-limited TaskGroup**: Combining a semaphore with a TaskGroup to cap the number of tasks running at once.
- **Producer-consumer**: A pattern where producers add work to a queue and consumers process it, all managed by a TaskGroup.

## Real World Context
An ETL pipeline needs to process 50,000 records from a database. Launching 50,000 concurrent tasks would exhaust memory. Instead, you batch them into groups of 100, use a semaphore to limit active database connections to 10, and wrap everything in a timeout so the pipeline fails fast if the database is unresponsive. Each of these patterns composes cleanly with TaskGroup.

## Deep Dive

### Batching with TaskGroup

```python
async def process_batch(items, batch_size=10):
    results = []
    
    for i in range(0, len(items), batch_size):
        batch = items[i:i + batch_size]
        
        async with asyncio.TaskGroup() as tg:
            tasks = [tg.create_task(process(item)) for item in batch]
        
        results.extend(task.result() for task in tasks)
    
    return results
```

### Timeout with TaskGroup

```python
async def with_timeout():
    try:
        async with asyncio.timeout(5.0):
            async with asyncio.TaskGroup() as tg:
                tg.create_task(slow_operation())
                tg.create_task(another_operation())
    except TimeoutError:
        print("Operations timed out")
        # All tasks automatically cancelled
```

### Semaphore-Limited TaskGroup

```python
async def limited_concurrent(urls, max_concurrent=5):
    semaphore = asyncio.Semaphore(max_concurrent)
    
    async def fetch_limited(url):
        async with semaphore:
            return await fetch(url)
    
    async with asyncio.TaskGroup() as tg:
        tasks = [tg.create_task(fetch_limited(url)) for url in urls]
    
    return [task.result() for task in tasks]
```

### Producer-Consumer with TaskGroup

```python
async def producer_consumer():
    queue = asyncio.Queue(maxsize=100)
    
    async def producer():
        for i in range(1000):
            await queue.put(i)
        # Signal completion
        for _ in range(num_consumers):
            await queue.put(None)
    
    async def consumer():
        while True:
            item = await queue.get()
            if item is None:
                break
            await process(item)
    
    num_consumers = 5
    async with asyncio.TaskGroup() as tg:
        tg.create_task(producer())
        for _ in range(num_consumers):
            tg.create_task(consumer())
```

## Common Pitfalls
1. **Launching too many tasks without batching** — Creating thousands of tasks at once can exhaust memory and overwhelm external services. Batch work into manageable groups.
2. **Putting the timeout inside the TaskGroup** — The timeout must wrap the TaskGroup, not the other way around. A timeout inside the group does not cancel sibling tasks on expiry.
3. **Forgetting sentinel values in producer-consumer** — Without a signal (like `None`) to tell consumers to stop, they will wait forever on the queue and the TaskGroup will never exit.

## Best Practices
1. **Compose patterns together** — Combine batching, semaphores, and timeouts as needed. For example, batch by 100, limit to 10 concurrent with a semaphore, and wrap in a 30-second timeout.
2. **Use `queue.join()` for completion tracking** — When using `task_done()` and `join()`, you get precise knowledge of when all items have been processed, not just when they have been dequeued.

## Summary
- Batch processing with TaskGroup controls memory usage by limiting the number of concurrent tasks per round.
- Wrapping a TaskGroup in `asyncio.timeout()` enforces a deadline and automatically cancels all tasks on expiry.
- Semaphore-limited TaskGroups cap concurrent access to rate-limited resources.
- Producer-consumer with TaskGroup ensures all producers and consumers have clean lifetimes.
- These patterns compose together for robust, production-ready async workflows.

## Code Examples

**Fallback pattern**

```python
# Retry pattern with TaskGroup
async def fetch_with_fallbacks(primary_url, fallback_urls):
    """Try primary, fall back to alternates."""
    async def try_fetch(url):
        result = await fetch(url)
        if result:
            return result
        raise FetchError(url)
    
    try:
        async with asyncio.TaskGroup() as tg:
            task = tg.create_task(try_fetch(primary_url))
        return task.result()
    except* FetchError:
        # Primary failed, try fallbacks
        async with asyncio.TaskGroup() as tg:
            tasks = [tg.create_task(try_fetch(url)) for url in fallback_urls]
        
        for task in tasks:
            if not task.cancelled() and task.exception() is None:
                return task.result()
        raise AllFetchesFailed()
```


## Resources

- [Coroutines and Tasks](https://docs.python.org/3.14/library/asyncio-task.html) — Advanced patterns for coroutines, tasks, and TaskGroups

---

> 📘 *This lesson is part of the [Python Concurrency: Asyncio & Free-Threading](https://stanza.dev/courses/python-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*