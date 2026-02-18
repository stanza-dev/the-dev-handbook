---
source_course: "python-concurrency"
source_lesson: "python-concurrency-ipc"
---

# Sharing Data Between Processes

## Queues

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

## Pipes

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

## Shared Values and Arrays

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

## Managers for Complex Objects

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


---

> 📘 *This lesson is part of the [Python Concurrency: Asyncio & No-GIL](https://stanza.dev/courses/python-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*