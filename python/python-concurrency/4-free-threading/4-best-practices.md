---
source_course: "python-concurrency"
source_lesson: "python-concurrency-nogil-best-practices"
---

# Best Practices

## 1. Prefer Immutable Data

```python
from dataclasses import dataclass

@dataclass(frozen=True)
class Config:
    host: str
    port: int

# Safe to share across threads
config = Config("localhost", 8080)
```

## 2. Use Thread-Safe Collections

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

## 3. Minimize Shared State

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

## 4. Use Higher-Level Abstractions

```python
from concurrent.futures import ThreadPoolExecutor

# Let the pool handle synchronization
with ThreadPoolExecutor(max_workers=4) as executor:
    results = list(executor.map(process, items))
```

## 5. Test with Race Detection

```python
# Use tools like:
# - ThreadSanitizer (TSan)
# - Python's faulthandler
# - Stress testing with many threads
```

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


---

> 📘 *This lesson is part of the [Python Concurrency: Asyncio & No-GIL](https://stanza.dev/courses/python-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*