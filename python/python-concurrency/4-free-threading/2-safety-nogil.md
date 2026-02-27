---
source_course: "python-concurrency"
source_lesson: "python-concurrency-thread-safety-nogil"
---

# Thread Safety in Free-Threading

## Introduction
With the GIL removed, Python threads can now truly run in parallel, which means true data races are possible for the first time in CPython's history. Code that appeared safe under the GIL (by accident, not by design) can now produce corrupted data. This lesson explains which operations are safe, which are not, and how to protect shared state.

## Key Concepts
- **Data race**: A bug where two or more threads access shared data concurrently and at least one modifies it, producing unpredictable results.
- **Atomic operation**: An operation that completes in a single step with respect to other threads. Other threads see either the state before or after, never in between.
- **Lock**: A synchronization primitive (`threading.Lock`) that ensures only one thread executes a critical section at a time.
- **Thread-local storage**: Per-thread data (`threading.local()`) where each thread sees its own independent copy.

## Real World Context
A web application maintains a global request counter. Under the GIL, `counter += 1` from multiple threads appeared to work (though it was technically unsafe). After migrating to free-threaded Python, the counter starts returning incorrect values because `+=` is a read-modify-write sequence that is not atomic. Wrapping it in a lock fixes the issue.

## Deep Dive

### Why Thread Safety Matters More

Without the GIL, true data races are possible:

```python
# DANGER: Race condition in free-threaded Python!
counter = 0

def increment():
    global counter
    for _ in range(1000000):
        counter += 1  # Not atomic!

threads = [threading.Thread(target=increment) for _ in range(4)]
for t in threads: t.start()
for t in threads: t.join()

# Expected: 4000000
# Actual (free-threaded): Some random lower number
```

### Atomic Operations

Some operations remain safe:

```python
# Safe in free-threaded Python:
list.append(item)     # Thread-safe
dict[key] = value     # Thread-safe
set.add(item)         # Thread-safe

# NOT safe:
counter += 1          # Read-modify-write
list[i] = list[i] + 1 # Multiple operations
```

### Using Locks

```python
import threading

lock = threading.Lock()
counter = 0

def safe_increment():
    global counter
    for _ in range(1000000):
        with lock:
            counter += 1
```

### Atomic Types (Future)

Python may add atomic types in future versions:

```python
# Hypothetical future API
from threading import AtomicInt

counter = AtomicInt(0)
counter.increment()  # Atomic operation
```

### Per-Thread Data

```python
import threading

# Thread-local storage
local = threading.local()

def worker():
    local.value = threading.current_thread().name
    # Each thread sees its own value
```

## Common Pitfalls
1. **Assuming the GIL made your code thread-safe** — Many Python programs had latent race conditions hidden by the GIL. Moving to free-threading exposes them. Audit all shared mutable state.
2. **Over-locking** — Wrapping large blocks of code in a single lock negates the benefits of parallelism. Lock only the critical section (the smallest possible region that accesses shared state).
3. **Confusing single-operation safety with multi-operation safety** — `list.append()` is atomic, but `if len(lst) > 0: lst.pop()` is not, because another thread can modify the list between the check and the pop.

## Best Practices
1. **Prefer thread-safe data structures** — Use `queue.Queue`, `collections.deque`, or `threading.local()` instead of raw shared state wherever possible.
2. **Use the `with lock:` context manager** — It guarantees the lock is released even if an exception occurs, preventing deadlocks from forgotten unlock calls.

## Summary
- Without the GIL, true data races are possible in Python for the first time.
- Simple single-operation mutations like `list.append()` and `dict[key] = value` remain thread-safe.
- Compound operations like `counter += 1` are not atomic and require explicit locking.
- Use `threading.Lock` with the `with` statement to protect critical sections.
- Thread-local storage (`threading.local()`) eliminates sharing entirely for per-thread data.

## Code Examples

**Thread-safe patterns**

```python
import threading
from queue import Queue

# Thread-safe data structures
results = Queue()  # Thread-safe queue

def worker(task_id):
    result = expensive_computation(task_id)
    results.put((task_id, result))  # Safe

threads = [threading.Thread(target=worker, args=(i,)) for i in range(10)]
for t in threads: t.start()
for t in threads: t.join()

# Collect results
while not results.empty():
    task_id, result = results.get()
    print(f"Task {task_id}: {result}")
```


## Resources

- [Free-threaded Python](https://docs.python.org/3.14/howto/free-threading-python.html) — Guide to thread safety and data races in free-threaded Python

---

> 📘 *This lesson is part of the [Python Concurrency: Asyncio & Free-Threading](https://stanza.dev/courses/python-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*