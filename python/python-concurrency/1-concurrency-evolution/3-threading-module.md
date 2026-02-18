---
source_course: "python-concurrency"
source_lesson: "python-concurrency-threading-module"
---

# Working with Threads

## Creating Threads

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

## Thread Properties and Methods

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

## Daemon Threads

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

## Thread-Local Data

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


---

> 📘 *This lesson is part of the [Python Concurrency: Asyncio & No-GIL](https://stanza.dev/courses/python-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*