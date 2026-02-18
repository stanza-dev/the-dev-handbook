---
source_course: "python-concurrency"
source_lesson: "python-concurrency-multiprocessing-basics"
---

# Multiprocessing Module

Multiprocessing creates separate Python processes, each with its own memory space and GIL.

## Why Multiprocessing?

- True parallelism for CPU-bound tasks
- Each process has its own memory (isolation)
- Works around GIL limitations
- Can utilize all CPU cores

## Creating Processes

```python
from multiprocessing import Process

def worker(name):
    print(f"Worker {name} starting")

if __name__ == '__main__':  # Required on Windows!
    p = Process(target=worker, args=('A',))
    p.start()
    p.join()  # Wait for completion
```

## Process vs Thread Trade-offs

| Aspect | Threading | Multiprocessing |
|--------|-----------|----------------|
| Memory | Shared | Isolated |
| Overhead | Low | Higher |
| Communication | Easy | Requires IPC |
| GIL | Affected | Bypassed |
| CPU-bound | Limited | Full parallelism |

## The `if __name__ == '__main__'` Guard

```python
# REQUIRED to prevent infinite process spawning on Windows
if __name__ == '__main__':
    main()
```

## Start Methods

```python
import multiprocessing as mp

# Different ways to start processes
mp.set_start_method('spawn')  # Default on Windows/macOS
mp.set_start_method('fork')   # Default on Linux (fast but unsafe)
mp.set_start_method('forkserver')  # Compromise

# Or use context
ctx = mp.get_context('spawn')
p = ctx.Process(target=worker)
```

## Code Examples

**Multiple processes**

```python
from multiprocessing import Process, current_process
import os

def worker():
    print(f"Process: {current_process().name}")
    print(f"PID: {os.getpid()}")
    print(f"Parent PID: {os.getppid()}")

if __name__ == '__main__':
    processes = [Process(target=worker, name=f"Worker-{i}")
                 for i in range(4)]
    
    for p in processes:
        p.start()
    
    for p in processes:
        p.join()
```


---

> 📘 *This lesson is part of the [Python Concurrency: Asyncio & No-GIL](https://stanza.dev/courses/python-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*