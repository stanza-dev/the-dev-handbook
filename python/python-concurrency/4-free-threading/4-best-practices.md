---
source_course: "python-concurrency"
source_lesson: "python-concurrency-nogil-best-practices"
---

# Best Practices for Free-Threading

## Introduction
Free-threaded Python gives you true parallelism with threads, but that power comes with responsibility. This lesson distills the most important practices for writing correct, performant free-threaded code: favoring immutability, using thread-safe collections, minimizing shared state, leveraging high-level abstractions, and testing for races.

## Key Concepts
- **Immutable data**: Data that cannot be modified after creation. Immutable objects are inherently thread-safe because no thread can change them.
- **Thread-safe collections**: Data structures designed to be safely accessed from multiple threads, such as `queue.Queue`.
- **Data partitioning**: Splitting work so each thread operates on its own slice, eliminating the need for synchronization.
- **Higher-level abstractions**: Tools like `ThreadPoolExecutor` that handle thread management and synchronization internally.

## Real World Context
A machine learning pipeline pre-processes training data across multiple threads. By partitioning the dataset so each thread works on its own chunk and using a `Queue` to collect results, the pipeline achieves near-linear speedup without a single lock. Frozen dataclasses carry configuration safely across threads. This approach scales from 4 to 64 cores without code changes.

## Deep Dive

### 1. Prefer Immutable Data

```python
from dataclasses import dataclass

@dataclass(frozen=True)
class Config:
    host: str
    port: int

# Safe to share across threads
config = Config("localhost", 8080)
```

### 2. Use Thread-Safe Collections

```python
from queue import Queue, LifoQueue, PriorityQueue
import threading

# Thread-safe
job_queue = Queue()
results = Queue()

# For simple counters, consider:
from collections import Counter
# Note: Counter itself isn't fully thread-safe
```

### 3. Minimize Shared State

```python
def parallel_process(items):
    # Each thread gets its own slice
    chunks = split_into_chunks(items, num_threads)
    
    results = []
    threads = []
    
    for chunk in chunks:
        t = threading.Thread(
            target=lambda c, r: r.extend(process_chunk(c)),
            args=(chunk, results)
        )
        threads.append(t)
        t.start()
    
    for t in threads:
        t.join()
    
    return results
```

### 4. Use Higher-Level Abstractions

```python
from concurrent.futures import ThreadPoolExecutor

# Let the pool handle synchronization
with ThreadPoolExecutor(max_workers=4) as executor:
    results = list(executor.map(process, items))
```

### 5. Test with Race Detection

```python
# Use tools like:
# - ThreadSanitizer (TSan)
# - Python's faulthandler
# - Stress testing with many threads
```

## Common Pitfalls
1. **Using mutable global state for configuration** — A shared mutable dict as config is a race condition waiting to happen. Use frozen dataclasses or `types.MappingProxyType` for read-only configuration.
2. **Collecting results in a shared list without protection** — `list.extend()` across threads can interleave and corrupt the list. Use a `Queue` to collect results from worker threads safely.
3. **Ignoring race detection tools** — Code that looks correct may have subtle races. ThreadSanitizer and stress tests catch issues that code review misses.

## Best Practices
1. **Design for immutability first** — If data does not need to change, make it immutable. Frozen dataclasses, tuples, and frozensets eliminate entire categories of thread-safety bugs.
2. **Partition work instead of sharing state** — Give each thread its own independent slice of the input. Merge results after all threads finish. This is the simplest path to correct parallel code.
3. **Use `ThreadPoolExecutor` or `InterpreterPoolExecutor`** — These high-level abstractions handle thread lifecycle and result collection, reducing the surface area for bugs.

## Summary
- Immutable data is inherently thread-safe and should be the default for shared configuration.
- Thread-safe collections like `Queue` are the safest way to pass data between threads.
- Data partitioning eliminates the need for synchronization by giving each thread its own slice.
- Higher-level abstractions like `ThreadPoolExecutor` manage threads and results safely.
- Always test with race detection tools and high thread counts.

## Code Examples

**Message-passing pattern**

```python
# Message-passing pattern (safer than shared state)
import threading
from queue import Queue

def worker(task_queue, result_queue):
    while True:
        task = task_queue.get()
        if task is None:
            break
        result = process(task)
        result_queue.put(result)

# Main thread controls all state
task_q = Queue()
result_q = Queue()

workers = [
    threading.Thread(target=worker, args=(task_q, result_q))
    for _ in range(4)
]

for w in workers:
    w.start()

# Send tasks
for task in tasks:
    task_q.put(task)

# Signal completion
for _ in workers:
    task_q.put(None)

for w in workers:
    w.join()
```


## Resources

- [Free-threaded Python](https://docs.python.org/3.14/howto/free-threading-python.html) — Best practices and patterns for writing free-threaded Python code

---

> 📘 *This lesson is part of the [Python Concurrency: Asyncio & Free-Threading](https://stanza.dev/courses/python-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*