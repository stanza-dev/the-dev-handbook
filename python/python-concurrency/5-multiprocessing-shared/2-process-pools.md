---
source_course: "python-concurrency"
source_lesson: "python-concurrency-process-pools"
---

# Process Pools

## Introduction
Managing individual processes is tedious for large workloads. Process pools maintain a fixed number of worker processes and distribute tasks automatically, handling process creation, communication, and cleanup. This lesson covers `multiprocessing.Pool`, its methods for synchronous and asynchronous work, chunking strategies, and the new Python 3.14 pool management features.

## Key Concepts
- **Pool**: A fixed-size collection of worker processes that execute tasks submitted by the main process.
- **map()**: Applies a function to every item in an iterable in parallel, returning results in order.
- **imap()**: A lazy, memory-efficient version of `map()` that yields results one at a time.
- **chunksize**: The number of items sent to each worker at once. Larger chunks reduce IPC overhead but increase memory usage.
- **apply_async()**: Submits a single task and returns immediately with an `AsyncResult` object.

## Real World Context
A data science team needs to apply a CPU-heavy feature extraction function to 100,000 images. Using `Pool(8).map(extract_features, images, chunksize=500)`, the work is distributed across 8 cores. The `chunksize` parameter reduces the overhead of sending one image at a time. For a streaming pipeline, `imap_unordered` yields results as they complete, keeping memory usage constant.

## Deep Dive

### Basic Pool Usage

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

### Pool Methods

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

### Chunking for Performance

```python
# Default: sends one item at a time (high overhead)
pool.map(func, range(10000))

# Better: send chunks
pool.map(func, range(10000), chunksize=100)
```

### imap for Memory Efficiency

```python
with Pool(4) as pool:
    # Returns iterator (lazy evaluation)
    for result in pool.imap(func, huge_list, chunksize=100):
        process(result)
    
    # Unordered for faster results
    for result in pool.imap_unordered(func, huge_list):
        process(result)  # Results may come out of order
```

### New Pool Management (Python 3.14+)

```python
with ProcessPoolExecutor(max_workers=4) as executor:
    futures = [executor.submit(task, i) for i in range(10)]
    
    # New in 3.14: forcefully stop workers
    executor.terminate_workers()  # Graceful termination
    executor.kill_workers()       # Immediate kill
```

## Common Pitfalls
1. **Using chunksize=1 (the default) for large iterables** — Sending one item at a time incurs massive IPC overhead. For 10,000 items, `chunksize=100` or higher dramatically reduces overhead.
2. **Forgetting that Pool requires picklable functions** — Top-level functions work, but lambdas, closures, and methods of local classes cannot be pickled and will fail.
3. **Not using the Pool as a context manager** — Without `with Pool() as pool:`, worker processes may not be properly terminated, leading to zombie processes.

## Best Practices
1. **Use `imap_unordered` for streaming results** — When you do not need results in order, `imap_unordered` returns results as soon as each worker finishes, reducing latency.
2. **Tune chunksize based on task duration** — For fast tasks (< 1ms), use large chunks (1000+). For slow tasks (> 1s), small chunks (1-10) are fine because IPC overhead is negligible relative to task time.
3. **Use the new `terminate_workers()` and `kill_workers()` in Python 3.14** — These give you explicit control over worker shutdown, useful for cleanup in long-running services.

## Summary
- Process pools manage a fixed set of worker processes and distribute tasks automatically.
- `pool.map()` parallelizes work across processes; `pool.imap()` does so lazily for memory efficiency.
- The `chunksize` parameter is critical for performance with large iterables.
- `apply_async()` submits individual tasks non-blockingly and returns `AsyncResult` objects.
- Python 3.14 adds `terminate_workers()` and `kill_workers()` for explicit pool management.

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


## Resources

- [multiprocessing.pool](https://docs.python.org/3.14/library/multiprocessing.html#module-multiprocessing.pool) — Pool of worker processes for parallel task execution

---

> 📘 *This lesson is part of the [Python Concurrency: Asyncio & Free-Threading](https://stanza.dev/courses/python-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*