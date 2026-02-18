---
source_course: "python-concurrency"
source_lesson: "python-concurrency-concurrent-futures"
---

# concurrent.futures Module

A high-level interface for asynchronously executing callables.

## ThreadPoolExecutor

```python
from concurrent.futures import ThreadPoolExecutor

def fetch(url):
    # Simulate network request
    return f"Data from {url}"

urls = ["url1", "url2", "url3", "url4"]

# Using context manager (auto-cleanup)
with ThreadPoolExecutor(max_workers=4) as executor:
    # Submit individual tasks
    future = executor.submit(fetch, urls[0])
    result = future.result()  # Blocks until done
    
    # Map function over inputs
    results = list(executor.map(fetch, urls))
```

## ProcessPoolExecutor

```python
from concurrent.futures import ProcessPoolExecutor

def cpu_intensive(n):
    return sum(i**2 for i in range(n))

# Uses multiple processes (bypasses GIL)
with ProcessPoolExecutor(max_workers=4) as executor:
    results = list(executor.map(cpu_intensive, [10**6] * 4))
```

## Working with Futures

```python
from concurrent.futures import ThreadPoolExecutor, as_completed

with ThreadPoolExecutor() as executor:
    futures = {executor.submit(fetch, url): url for url in urls}
    
    # Process results as they complete
    for future in as_completed(futures):
        url = futures[future]
        try:
            result = future.result()
            print(f"{url}: {result}")
        except Exception as e:
            print(f"{url} failed: {e}")
```

## Future Methods

```python
future = executor.submit(task)

future.result(timeout=5)  # Get result (blocks)
future.done()             # Check if complete
future.cancelled()        # Check if cancelled
future.cancel()           # Attempt to cancel
future.add_done_callback(fn)  # Callback when done
```

## Code Examples

**Advanced future handling**

```python
from concurrent.futures import ThreadPoolExecutor, wait, FIRST_COMPLETED

with ThreadPoolExecutor() as executor:
    futures = [executor.submit(task, i) for i in range(10)]
    
    # Wait for first completion
    done, not_done = wait(futures, return_when=FIRST_COMPLETED)
    
    # Wait for all with timeout
    done, not_done = wait(futures, timeout=5)
    
    # Cancel remaining
    for f in not_done:
        f.cancel()
```


---

> 📘 *This lesson is part of the [Python Concurrency: Asyncio & No-GIL](https://stanza.dev/courses/python-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*