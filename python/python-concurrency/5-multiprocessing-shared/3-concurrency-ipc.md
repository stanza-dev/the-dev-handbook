---
source_course: "python-concurrency"
source_lesson: "python-concurrency-ipc"
---

# Inter-Process Communication

## Introduction
Since processes have isolated memory spaces, you cannot simply share variables between them. Data must be explicitly sent via inter-process communication (IPC) mechanisms. Python's `multiprocessing` module provides several options, from high-level queues and managers to low-level pipes and shared memory. This lesson covers each IPC mechanism and when to use it.

## Key Concepts
- **Queue**: A process-safe FIFO queue for sending picklable objects between processes.
- **Pipe**: A two-way communication channel between exactly two processes, faster than Queue for point-to-point communication.
- **Value / Array**: Shared memory wrappers for single values or arrays of a fixed C type.
- **Manager**: A server process that hosts Python objects (lists, dicts, namespaces) and proxies access from other processes.

## Real World Context
A real-time data processing system has producer processes reading from sensors and consumer processes running analytics. A `multiprocessing.Queue` connects them, decoupling the producers from the consumers. For high-performance numerical work, `Value` and `Array` share counters and buffers directly in shared memory without the overhead of pickling.

## Deep Dive

### Queues

```python
from multiprocessing import Process, Queue

def producer(queue):
    for i in range(5):
        queue.put(i)
    queue.put(None)  # Signal completion

def consumer(queue):
    while True:
        item = queue.get()
        if item is None:
            break
        print(f"Consumed: {item}")

if __name__ == '__main__':
    q = Queue()
    p1 = Process(target=producer, args=(q,))
    p2 = Process(target=consumer, args=(q,))
    p1.start(); p2.start()
    p1.join(); p2.join()
```

### Pipes

```python
from multiprocessing import Pipe

def sender(conn):
    conn.send({'data': [1, 2, 3]})
    conn.close()

def receiver(conn):
    data = conn.recv()
    print(f"Received: {data}")

parent_conn, child_conn = Pipe()
```

### Shared Values and Arrays

```python
from multiprocessing import Value, Array

# Shared counter (use with lock!)
counter = Value('i', 0)  # 'i' = integer

with counter.get_lock():
    counter.value += 1

# Shared array
arr = Array('d', [1.0, 2.0, 3.0])  # 'd' = double
arr[0] = 99.0
```

### Managers for Complex Objects

```python
from multiprocessing import Manager

if __name__ == '__main__':
    with Manager() as manager:
        shared_list = manager.list()
        shared_dict = manager.dict()
        shared_ns = manager.Namespace()
        
        # Pass to processes
        p = Process(target=worker, args=(shared_list,))
```

## Common Pitfalls
1. **Using Value without its lock** — `Value` objects have a built-in lock, but compound operations still require `with counter.get_lock():`. Without it, read-modify-write operations are racy.
2. **Deadlocking with Pipes** — If both ends of a Pipe try to send without anyone receiving, the internal buffer fills up and both processes block forever. Always pair sends with receives.
3. **Overusing Managers for performance-critical data** — Manager proxies route every access through a server process, adding significant latency. Use `shared_memory` or `Value/Array` for high-throughput numerical data.

## Best Practices
1. **Use Queue for most IPC needs** — It handles serialization, synchronization, and buffering. It is the safest and most flexible IPC mechanism for general use.
2. **Use Value/Array for simple shared counters and buffers** — They avoid pickling overhead and provide direct shared memory access, but require careful locking for compound operations.

## Summary
- Processes have isolated memory; data sharing requires explicit IPC mechanisms.
- `Queue` is the most versatile option: process-safe, supports multiple producers and consumers, and handles serialization automatically.
- `Pipe` is faster for two-process communication but does not scale to multiple processes.
- `Value` and `Array` provide shared memory for simple C types, requiring manual locking for safety.
- `Manager` proxies complex Python objects across processes at the cost of performance.

## Code Examples

**Task queue pattern**

```python
from multiprocessing import Process, Queue
import time

def worker(task_queue, result_queue):
    while True:
        task = task_queue.get()
        if task is None:
            break
        result = task ** 2  # Computation
        result_queue.put(result)

if __name__ == '__main__':
    tasks = Queue()
    results = Queue()
    
    # Start workers
    workers = [Process(target=worker, args=(tasks, results))
               for _ in range(4)]
    for w in workers:
        w.start()
    
    # Submit tasks
    for i in range(100):
        tasks.put(i)
    
    # Signal workers to stop
    for _ in workers:
        tasks.put(None)
    
    # Collect results
    output = [results.get() for _ in range(100)]
    
    for w in workers:
        w.join()
```


## Resources

- [Pipes and Queues](https://docs.python.org/3.14/library/multiprocessing.html#pipes-and-queues) — Inter-process communication using pipes and queues

---

> 📘 *This lesson is part of the [Python Concurrency: Asyncio & Free-Threading](https://stanza.dev/courses/python-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*