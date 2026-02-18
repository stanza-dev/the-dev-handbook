---
source_course: "python-concurrency"
source_lesson: "python-concurrency-threads-processes-gil"
---

# The Global Interpreter Lock (GIL)

## What is the GIL?

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

## Why Does the GIL Exist?

1. **Simplifies memory management** - Reference counting is not thread-safe
2. **Protects C extensions** - Many C libraries aren't thread-safe
3. **Single-threaded performance** - Actually faster for single-threaded code

## GIL Implications

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

## When Threads DO Help

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

> 📘 *This lesson is part of the [Python Concurrency: Asyncio & No-GIL](https://stanza.dev/courses/python-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*