---
source_course: "python-concurrency"
source_lesson: "python-concurrency-process-pools"
---

# Pool for Parallel Execution

Pools manage a fixed number of worker processes and distribute tasks.

## Basic Pool Usage

```python
from multiprocessing import Pool

def square(x):
    return x ** 2

if __name__ == '__main__':
    with Pool(4) as pool:  # 4 worker processes
        # map: apply function to all items
        results = pool.map(square, range(10))
        print(results)  # [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
```

## Pool Methods

```python
with Pool(4) as pool:
    # Synchronous methods
    results = pool.map(func, items)        # Returns list
    results = pool.starmap(func, [(a,b)])  # Unpack args
    
    # Async methods (return immediately)
    result = pool.apply_async(func, args)  # Single call
    async_map = pool.map_async(func, items)  # Parallel map
    
    # Get async results
    value = result.get(timeout=30)
    values = async_map.get()
```

## Chunking for Performance

```python
# Default: sends one item at a time (high overhead)
pool.map(func, range(10000))

# Better: send chunks
pool.map(func, range(10000), chunksize=100)
```

## imap for Memory Efficiency

```python
with Pool(4) as pool:
    # Returns iterator (lazy evaluation)
    for result in pool.imap(func, huge_list, chunksize=100):
        process(result)
    
    # Unordered for faster results
    for result in pool.imap_unordered(func, huge_list):
        process(result)  # Results may come out of order
```

## Code Examples

**Pool speedup example**

```python
from multiprocessing import Pool
import time

def slow_task(x):
    time.sleep(0.1)
    return x ** 2

if __name__ == '__main__':
    items = list(range(100))
    
    # Serial
    start = time.time()
    serial_results = [slow_task(x) for x in items]
    print(f"Serial: {time.time() - start:.2f}s")
    
    # Parallel
    start = time.time()
    with Pool(4) as pool:
        parallel_results = pool.map(slow_task, items)
    print(f"Parallel: {time.time() - start:.2f}s")
```


---

> 📘 *This lesson is part of the [Python Concurrency: Asyncio & No-GIL](https://stanza.dev/courses/python-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*