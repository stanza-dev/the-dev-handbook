---
source_course: "python-concurrency"
source_lesson: "python-concurrency-tasks-gather"
---

# Creating and Managing Tasks

## asyncio.create_task()

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

## asyncio.gather()

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

## asyncio.wait()

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

## Task Cancellation

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


---

> 📘 *This lesson is part of the [Python Concurrency: Asyncio & No-GIL](https://stanza.dev/courses/python-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*