---
source_course: "python-concurrency"
source_lesson: "python-concurrency-threads-processes-gil"
---

# Threads, Processes & The GIL

## Introduction
The Global Interpreter Lock is the single most important concept to understand before writing concurrent Python. It explains why multi-threaded Python code sometimes runs *slower* than single-threaded code, and it motivates the existence of multiprocessing and the new free-threading mode. This lesson breaks down what the GIL is, why it exists, and when it does and does not matter.

## Key Concepts
- **GIL (Global Interpreter Lock)**: A mutex in CPython that allows only one thread to execute Python bytecode at a time.
- **Reference counting**: CPython's primary garbage collection mechanism, which is not thread-safe without the GIL.
- **I/O-bound vs CPU-bound**: The GIL is released during I/O operations, so threads help with I/O; they do not help with pure computation.

## Real World Context
A team builds a web server that spawns a thread per request. For serving web pages (I/O-bound), it works well because the GIL is released during socket operations. But when they add an image-processing endpoint (CPU-bound), throughput collapses because only one thread can run the processing code at a time. Understanding the GIL would have led them to use a process pool for the CPU work.

## Deep Dive

### What is the GIL?

The GIL is a mutex that protects access to Python objects, preventing multiple native threads from executing Python bytecodes simultaneously.

```python
# Even with multiple threads, only one runs Python code at a time
import threading

counter = 0

def increment():
    global counter
    for _ in range(1000000):
        counter += 1  # Not atomic!

# Race condition possible despite GIL!
```

### Why Does the GIL Exist?

1. **Simplifies memory management** - Reference counting is not thread-safe
2. **Protects C extensions** - Many C libraries aren't thread-safe
3. **Single-threaded performance** - Actually faster for single-threaded code

### GIL Implications

```python
import threading
import time

def cpu_bound():
    count = 0
    for i in range(10**7):
        count += 1

# Serial execution
start = time.time()
cpu_bound()
cpu_bound()
print(f"Serial: {time.time() - start:.2f}s")

# Threaded execution (NOT faster with GIL!)
start = time.time()
t1 = threading.Thread(target=cpu_bound)
t2 = threading.Thread(target=cpu_bound)
t1.start(); t2.start()
t1.join(); t2.join()
print(f"Threaded: {time.time() - start:.2f}s")  # Similar or slower!
```

### When Threads DO Help

The GIL is released during I/O operations:

```python
import threading
import urllib.request

def fetch(url):
    urllib.request.urlopen(url).read()  # GIL released during I/O

# Multiple downloads run concurrently
threads = [threading.Thread(target=fetch, args=(url,)) for url in urls]
for t in threads: t.start()
for t in threads: t.join()
```

## Common Pitfalls
1. **Assuming the GIL makes code thread-safe** — The GIL prevents simultaneous bytecode execution, but compound operations like `counter += 1` involve multiple bytecodes and can still produce race conditions.
2. **Using threads for CPU-bound speedup** — Adding more threads to a CPU-bound task will not improve performance and may make it worse due to GIL contention overhead.

## Best Practices
1. **Use multiprocessing or free-threading for CPU-bound parallelism** — Each process has its own GIL, and free-threading removes it entirely, both enabling true parallel computation.
2. **Always protect shared mutable state with locks** — Even under the GIL, compound operations are not atomic. Use `threading.Lock` to guard critical sections.

## Summary
- The GIL allows only one thread to execute Python bytecode at a time in standard CPython.
- It exists to simplify memory management via reference counting and to protect C extensions.
- Threads still help with I/O-bound tasks because the GIL is released during I/O operations.
- For CPU-bound parallelism, use multiprocessing or free-threaded Python.
- The GIL does not make your code thread-safe; you still need locks for shared mutable state.

## Code Examples

**Thread synchronization**

```python
import threading

# Thread-safe operations
lock = threading.Lock()
counter = 0

def safe_increment():
    global counter
    with lock:  # Acquire lock
        temp = counter
        temp += 1
        counter = temp
    # Lock automatically released

# Thread synchronization primitives
event = threading.Event()      # Signal between threads
semaphore = threading.Semaphore(3)  # Limit concurrent access
barrier = threading.Barrier(4)  # Wait for all threads
```


## Resources

- [PEP 703](https://peps.python.org/pep-0703/) — Making the Global Interpreter Lock Optional in CPython

---

> 📘 *This lesson is part of the [Python Concurrency: Asyncio & Free-Threading](https://stanza.dev/courses/python-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*