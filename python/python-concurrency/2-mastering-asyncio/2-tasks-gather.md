---
source_course: "python-concurrency"
source_lesson: "python-concurrency-tasks-gather"
---

# Tasks and Gathering Results

## Introduction
Creating tasks and gathering their results is how you unlock the power of concurrency in asyncio. Instead of awaiting coroutines one by one (which runs them sequentially), you schedule them as tasks that run concurrently. This lesson shows you how to create tasks, collect results, handle errors, and cancel work that is no longer needed.

## Key Concepts
- **Task**: A wrapper around a coroutine that schedules it for concurrent execution on the event loop.
- **asyncio.create_task()**: Converts a coroutine into a Task and starts running it immediately.
- **asyncio.gather()**: Runs multiple coroutines concurrently and returns all results in order.
- **asyncio.wait()**: Provides fine-grained control, letting you react when the first task completes or after a timeout.
- **Cancellation**: Tasks can be cancelled, which raises `asyncio.CancelledError` inside the coroutine.

## Real World Context
A dashboard page needs to load user profile data, recent activity, and notification counts from three different API endpoints. Using `asyncio.gather()`, all three requests run concurrently and the total wait time equals the slowest request, not the sum of all three. Without gather, a page that calls three 200ms endpoints takes 600ms instead of 200ms.

## Deep Dive

### asyncio.create_task()

Convert a coroutine to a Task for concurrent execution:

```python
async def main():
    # Create tasks (starts running immediately)
    task1 = asyncio.create_task(fetch_data("url1"))
    task2 = asyncio.create_task(fetch_data("url2"))
    
    # Do other work while tasks run...
    
    # Wait for results
    result1 = await task1
    result2 = await task2
```

### asyncio.gather()

Run multiple coroutines concurrently and collect results:

```python
async def main():
    results = await asyncio.gather(
        fetch_data("url1"),
        fetch_data("url2"),
        fetch_data("url3"),
    )
    # results is a list: [result1, result2, result3]
```

### Error Handling with gather

```python
results = await asyncio.gather(
    task1(),
    task2(),
    return_exceptions=True  # Don't raise, return exceptions
)

for result in results:
    if isinstance(result, Exception):
        print(f"Task failed: {result}")
    else:
        print(f"Success: {result}")
```

### asyncio.wait()

More control over completion:

```python
done, pending = await asyncio.wait(
    tasks,
    timeout=5.0,
    return_when=asyncio.FIRST_COMPLETED  # or ALL_COMPLETED
)

for task in done:
    result = task.result()

for task in pending:
    task.cancel()
```

### Task Cancellation

```python
task = asyncio.create_task(long_operation())

# Cancel after 5 seconds
await asyncio.sleep(5)
task.cancel()

try:
    await task
except asyncio.CancelledError:
    print("Task was cancelled")
```

## Common Pitfalls
1. **Fire-and-forget tasks** — Calling `asyncio.create_task(work())` without storing a reference means the task can be garbage-collected before it completes. Always keep a reference and eventually `await` it.
2. **Ignoring return_exceptions in gather** — Without `return_exceptions=True`, `gather` raises the first exception and you lose results from other tasks. Use it when partial success is acceptable.
3. **Not handling CancelledError** — When a task is cancelled, `CancelledError` propagates. If you catch `Exception` broadly, you may accidentally swallow cancellation signals.

## Best Practices
1. **Prefer TaskGroup over gather for new code** — `asyncio.TaskGroup` (Python 3.11+) provides structured concurrency with automatic cancellation on failure. Use gather mainly for backwards compatibility.
2. **Always cancel pending tasks on timeout** — When using `asyncio.wait()` with a timeout, explicitly cancel remaining tasks to avoid orphaned work.

## Summary
- `asyncio.create_task()` schedules a coroutine for concurrent execution and returns a Task.
- `asyncio.gather()` runs multiple coroutines concurrently and collects all results in order.
- `asyncio.wait()` gives fine-grained control with FIRST_COMPLETED and timeout support.
- Always store references to tasks and handle `CancelledError` for clean shutdowns.
- Use `return_exceptions=True` in `gather` when you want partial results instead of an immediate exception.

## Code Examples

**Concurrent fetching with gather**

```python
import asyncio

async def fetch(url, delay):
    await asyncio.sleep(delay)
    return f"Data from {url}"

async def main():
    # Run concurrently
    results = await asyncio.gather(
        fetch("api/users", 1),
        fetch("api/posts", 2),
        fetch("api/comments", 0.5),
    )
    
    # Results in order: [users, posts, comments]
    # Total time: ~2 seconds (not 3.5!)
    for r in results:
        print(r)

asyncio.run(main())
```


## Resources

- [Coroutines and Tasks](https://docs.python.org/3.14/library/asyncio-task.html) — Creating and managing asyncio tasks, gather, and wait

---

> 📘 *This lesson is part of the [Python Concurrency: Asyncio & Free-Threading](https://stanza.dev/courses/python-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*