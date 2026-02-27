---
source_course: "python-concurrency"
source_lesson: "python-concurrency-concurrent-futures"
---

# concurrent.futures: High-Level Interface

## Introduction
The `concurrent.futures` module provides a clean, high-level API for running tasks in parallel using thread or process pools. Instead of manually creating threads, managing their lifecycle, and collecting results, you submit callables to an executor and work with Future objects. This lesson covers both classic executors and the new Python 3.14 `InterpreterPoolExecutor`.

## Key Concepts
- **Executor**: An object that manages a pool of workers (threads or processes) and distributes tasks to them.
- **Future**: A placeholder for a result that has not been computed yet. You can check its status, wait for it, or attach callbacks.
- **ThreadPoolExecutor**: An executor backed by a thread pool, ideal for I/O-bound work.
- **ProcessPoolExecutor**: An executor backed by a process pool, ideal for CPU-bound work.
- **InterpreterPoolExecutor** (Python 3.14+): An executor using subinterpreters for parallel execution with lower overhead than processes.

## Real World Context
A data pipeline needs to download 500 CSV files from an API, then parse each one. With `concurrent.futures`, you use a `ThreadPoolExecutor` for the downloads (I/O-bound) and swap to a `ProcessPoolExecutor` for the parsing (CPU-bound), all with the same simple `executor.map()` interface. The abstraction lets you switch strategies without rewriting your task logic.

## Deep Dive

### ThreadPoolExecutor

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

### ProcessPoolExecutor

```python
from concurrent.futures import ProcessPoolExecutor

def cpu_intensive(n):
    return sum(i**2 for i in range(n))

# Uses multiple processes (bypasses GIL)
with ProcessPoolExecutor(max_workers=4) as executor:
    results = list(executor.map(cpu_intensive, [10**6] * 4))
```

### Working with Futures

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

### Future Methods

```python
future = executor.submit(task)

future.result(timeout=5)  # Get result (blocks)
future.done()             # Check if complete
future.cancelled()        # Check if cancelled
future.cancel()           # Attempt to cancel
future.add_done_callback(fn)  # Callback when done
```

### InterpreterPoolExecutor (Python 3.14+)

Python 3.14 introduces `InterpreterPoolExecutor` which uses subinterpreters instead of threads or processes:

```python
from concurrent.futures import InterpreterPoolExecutor

def cpu_task(n):
    return sum(i**2 for i in range(n))

with InterpreterPoolExecutor(max_workers=4) as executor:
    results = list(executor.map(cpu_task, [10**6] * 4))
```

Subinterpreters have their own GIL, enabling true parallelism like multiprocessing, but with lower overhead since they share the same OS process.

### concurrent.interpreters Module (Python 3.14+)

```python
from concurrent import interpreters

interp = interpreters.create()
interp.exec('print("Hello from subinterpreter!")')

# Call functions across interpreters
# Note: functions and arguments must be picklable.
# Lambdas, closures, and local classes cannot be passed.
def compute(x):
    return x ** 2

result = interp.call(compute, 42)  # Returns 1764
```

## Common Pitfalls
1. **Not handling exceptions from futures** — If a task raises an exception, it is stored silently in the Future. If you never call `future.result()`, you will never see the error. Always check results or use `as_completed()` with try/except.
2. **Setting max_workers too high** — More workers does not always mean faster. Too many threads cause excessive context switching; too many processes exhaust system memory. A good default for threads is `min(32, os.cpu_count() + 4)`.
3. **Using ProcessPoolExecutor with unpicklable arguments** — Process pools serialize arguments with pickle. Lambdas, closures, and certain objects cannot be pickled and will raise errors.

## Best Practices
1. **Always use executors as context managers** — The `with` statement ensures workers are properly shut down and resources are released, even if an exception occurs.
2. **Use `as_completed()` for responsiveness** — When tasks have varying durations, `as_completed()` lets you process results as they arrive rather than waiting for all tasks to finish.

## Summary
- `concurrent.futures` provides a unified high-level API for thread and process pools.
- `ThreadPoolExecutor` is best for I/O-bound tasks; `ProcessPoolExecutor` is best for CPU-bound tasks.
- `InterpreterPoolExecutor` (Python 3.14) offers parallel execution via subinterpreters with lower overhead than processes.
- Always use context managers for executors and handle exceptions from Future objects.
- The `as_completed()` function processes results in completion order for better responsiveness.

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


## Resources

- [concurrent.futures Module](https://docs.python.org/3.14/library/concurrent.futures.html) — High-level interface for asynchronously executing callables

---

> 📘 *This lesson is part of the [Python Concurrency: Asyncio & Free-Threading](https://stanza.dev/courses/python-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*