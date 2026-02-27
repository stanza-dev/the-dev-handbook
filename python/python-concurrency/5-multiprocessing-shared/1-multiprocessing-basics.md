---
source_course: "python-concurrency"
source_lesson: "python-concurrency-multiprocessing-basics"
---

# Multiprocessing Basics

## Introduction
The `multiprocessing` module creates separate Python processes, each with its own memory space and GIL, enabling true parallelism for CPU-bound work. Unlike threading, multiprocessing is not limited by the GIL and can fully utilize all CPU cores. This lesson covers process creation, the key trade-offs versus threading, start methods, and the Python 3.14 default change.

## Key Concepts
- **Process**: An independent instance of the Python interpreter with its own memory space and GIL.
- **Start method**: How new processes are created: `spawn` (safe, default on Windows/macOS), `fork` (fast but unsafe with threads), or `forkserver` (compromise).
- **`if __name__ == '__main__'` guard**: Required on Windows and macOS to prevent infinite process spawning when the module is reimported by child processes.
- **IPC (Inter-Process Communication)**: Since processes do not share memory by default, data must be explicitly sent between them via queues, pipes, or shared memory.

## Real World Context
A video transcoding service needs to process multiple video files simultaneously. Each transcode is pure CPU work, so threading with the GIL provides no speedup. Using `multiprocessing.Process`, each video is transcoded in a separate process on its own core. The main process distributes files and collects results via a `Queue`.

## Deep Dive

### Why Multiprocessing?

- True parallelism for CPU-bound tasks
- Each process has its own memory (isolation)
- Works around GIL limitations
- Can utilize all CPU cores

### Creating Processes

```python
from multiprocessing import Process

def worker(name):
    print(f"Worker {name} starting")

if __name__ == '__main__':  # Required on Windows!
    p = Process(target=worker, args=('A',))
    p.start()
    p.join()  # Wait for completion
```

### Process vs Thread Trade-offs

| Aspect | Threading | Multiprocessing |
|--------|-----------|----------------|
| Memory | Shared | Isolated |
| Overhead | Low | Higher |
| Communication | Easy | Requires IPC |
| GIL | Affected | Bypassed |
| CPU-bound | Limited | Full parallelism |

### The `if __name__ == '__main__'` Guard

```python
# REQUIRED to prevent infinite process spawning on Windows
if __name__ == '__main__':
    main()
```

### Start Methods

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

### Default Start Method Change (Python 3.14)

In Python 3.14, the default start method on Unix (except macOS) changed from `'fork'` to `'forkserver'`. This is safer because `fork` can cause issues with threads and locks. If your code depends on `'fork'`, explicitly set it:

```python
mp.set_start_method('fork')  # Explicit is better than implicit
```

## Common Pitfalls
1. **Forgetting the `if __name__ == '__main__'` guard** — On Windows and macOS (with `spawn`), omitting this guard causes each child process to re-execute the module, spawning infinite processes and crashing the system.
2. **Passing unpicklable objects to processes** — Arguments and return values are serialized with pickle. Lambdas, open file handles, and database connections cannot be pickled and will raise errors.
3. **Assuming shared memory by default** — Unlike threads, processes have isolated memory. Modifying a variable in a child process does not affect the parent. Use `Queue`, `Value`, or `shared_memory` for communication.

## Best Practices
1. **Explicitly set your start method** — Do not rely on platform defaults, especially with the Python 3.14 change from `fork` to `forkserver`. Explicitly call `mp.set_start_method()` at the top of your script.
2. **Prefer `spawn` for safety** — The `spawn` start method is the safest choice because it starts a fresh interpreter without inheriting locks or threads from the parent process.

## Summary
- `multiprocessing` creates separate Python processes for true CPU-bound parallelism.
- Each process has its own memory space and GIL, bypassing the GIL limitation entirely.
- The `if __name__ == '__main__'` guard is required to prevent infinite process spawning.
- Python 3.14 changed the default Unix start method from `fork` to `forkserver` for safety.
- Communication between processes requires explicit IPC mechanisms like queues and shared memory.

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


## Resources

- [multiprocessing Module](https://docs.python.org/3.14/library/multiprocessing.html) — Process-based parallelism with the multiprocessing module

---

> 📘 *This lesson is part of the [Python Concurrency: Asyncio & Free-Threading](https://stanza.dev/courses/python-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*