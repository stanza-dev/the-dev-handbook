---
source_course: "python-concurrency"
source_lesson: "python-concurrency-thread-safety-nogil"
---

# Thread Safety Without the GIL

## Why Thread Safety Matters More

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

## Atomic Operations

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

## Using Locks

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

## Atomic Types (Future)

Python may add atomic types in future versions:

```python
# Hypothetical future API
from threading import AtomicInt

counter = AtomicInt(0)
counter.increment()  # Atomic operation
```

## Per-Thread Data

```python
import threading

# Thread-local storage
local = threading.local()

def worker():
    local.value = threading.current_thread().name
    # Each thread sees its own value
```

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


---

> 📘 *This lesson is part of the [Python Concurrency: Asyncio & No-GIL](https://stanza.dev/courses/python-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*