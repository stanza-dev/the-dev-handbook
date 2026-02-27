---
source_course: "python-concurrency"
source_lesson: "python-concurrency-threading-module"
---

# The threading Module

## Introduction
Python's `threading` module is the standard way to run tasks concurrently within a single process. Whether you are handling multiple network connections, running background tasks, or building a producer-consumer pipeline, threads are a fundamental building block. This lesson walks through creating, managing, and coordinating threads.

## Key Concepts
- **Thread**: A lightweight unit of execution that shares memory with other threads in the same process.
- **Daemon thread**: A background thread that is automatically terminated when the main program exits.
- **Thread-local data**: Storage where each thread sees its own independent copy of a variable.
- **join()**: A method that blocks the calling thread until the target thread finishes.

## Real World Context
A chat server needs to handle hundreds of simultaneous client connections. Each connection can be managed in its own thread, reading messages and broadcasting them. Daemon threads handle periodic tasks like logging stats, and thread-local data stores per-connection context without passing it through every function call.

## Deep Dive

### Creating Threads

```python
import threading

# Method 1: Pass a function
def worker(name):
    print(f"Worker {name} starting")

t = threading.Thread(target=worker, args=("A",))
t.start()
t.join()  # Wait for completion

# Method 2: Subclass Thread
class MyThread(threading.Thread):
    def __init__(self, name):
        super().__init__()
        self.name = name
    
    def run(self):
        print(f"Thread {self.name} running")

t = MyThread("B")
t.start()
```

### Thread Properties and Methods

```python
t = threading.Thread(target=worker)

t.start()       # Start the thread
t.join(timeout=5)  # Wait for completion
t.is_alive()    # Check if running
t.name          # Thread name
t.daemon = True # Daemon thread (dies with main)

threading.current_thread()  # Get current thread
threading.active_count()    # Number of active threads
threading.enumerate()       # List all threads
```

### Daemon Threads

```python
import threading
import time

def background_task():
    while True:
        print("Background work...")
        time.sleep(1)

# Daemon thread dies when main program exits
t = threading.Thread(target=background_task, daemon=True)
t.start()

print("Main program continues...")
time.sleep(3)
print("Main program exits, daemon dies")
```

### Thread-Local Data

```python
import threading

# Each thread gets its own copy
local_data = threading.local()

def worker(value):
    local_data.x = value  # Thread-specific
    print(f"Thread sees x = {local_data.x}")

threads = [threading.Thread(target=worker, args=(i,)) for i in range(3)]
for t in threads: t.start()
for t in threads: t.join()
```

## Common Pitfalls
1. **Forgetting to call join()** — If the main thread exits before worker threads finish, daemon threads are killed immediately and non-daemon threads keep the process alive unexpectedly. Always join threads you care about.
2. **Subclassing Thread without calling super().__init__()** — Forgetting to call the parent constructor in a Thread subclass causes cryptic errors when you try to start the thread.
3. **Sharing mutable objects without locks** — Threads share memory. Without synchronization, concurrent writes to the same data structure produce race conditions.

## Best Practices
1. **Prefer passing functions over subclassing Thread** — Using `Thread(target=fn)` is simpler and more Pythonic. Reserve subclassing for cases where you need to store complex per-thread state.
2. **Use daemon threads for background work that can be interrupted** — Daemon threads are ideal for logging, heartbeats, or monitoring tasks that should stop when the main program ends.

## Summary
- Create threads by passing a callable to `threading.Thread(target=fn)` or by subclassing `Thread`.
- Use `join()` to wait for a thread to complete and `daemon=True` for background threads.
- Thread-local data gives each thread its own isolated copy of a variable.
- Always synchronize access to shared mutable state with locks.

## Code Examples

**Producer-consumer with threads**

```python
import threading
from queue import Queue

# Producer-Consumer pattern
def producer(queue):
    for i in range(5):
        queue.put(i)
        print(f"Produced {i}")

def consumer(queue):
    while True:
        item = queue.get()
        if item is None:
            break
        print(f"Consumed {item}")
        queue.task_done()

q = Queue()
p = threading.Thread(target=producer, args=(q,))
c = threading.Thread(target=consumer, args=(q,))

p.start(); c.start()
p.join()
q.put(None)  # Signal consumer to stop
c.join()
```


## Resources

- [threading Module](https://docs.python.org/3.14/library/threading.html) — Official documentation for Python's threading module

---

> 📘 *This lesson is part of the [Python Concurrency: Asyncio & Free-Threading](https://stanza.dev/courses/python-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*