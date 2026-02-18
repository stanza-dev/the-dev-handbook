---
source_course: "python-concurrency"
source_lesson: "python-concurrency-no-gil-explained"
---

# The End of the GIL

## What is Free-Threading?

Free-threaded Python (PEP 703) removes the Global Interpreter Lock, allowing multiple OS threads to execute Python bytecode truly in parallel.

## The Evolution

- **Python 3.13**: Experimental free-threading support
- **Python 3.14**: Improved stability, more libraries compatible
- **Python 3.15**: Further refinements, broader adoption

## Performance Characteristics

| Scenario | GIL Python | Free-Threaded |
|----------|------------|---------------|
| Single-threaded | Baseline | ~5-10% slower |
| Multi-threaded CPU | No speedup | Near-linear scaling |
| Multi-threaded I/O | Good | Similar |

## Checking Your Build

```python
import sys

# Check if free-threading is available
if hasattr(sys, '_is_gil_enabled'):
    if sys._is_gil_enabled():
        print("GIL is enabled")
    else:
        print("Free-threaded mode!")
else:
    print("Standard GIL Python")

# Python version info
print(sys.version)  # Shows 't' suffix for free-threaded
```

## Installing Free-Threaded Python

```bash
# Using pyenv
pyenv install 3.13t  # 't' suffix for free-threaded

# Using conda
conda create -n freethreaded python=3.13 python_abi=cp313t
```

## Code Examples

**Comparing serial vs parallel**

```python
import sys
import threading
import time

def cpu_task():
    total = 0
    for i in range(10_000_000):
        total += i
    return total

# Single-threaded timing
start = time.perf_counter()
for _ in range(4):
    cpu_task()
serial_time = time.perf_counter() - start

# Multi-threaded timing
start = time.perf_counter()
threads = [threading.Thread(target=cpu_task) for _ in range(4)]
for t in threads: t.start()
for t in threads: t.join()
parallel_time = time.perf_counter() - start

print(f"Serial: {serial_time:.2f}s")
print(f"Parallel: {parallel_time:.2f}s")
print(f"Speedup: {serial_time/parallel_time:.2f}x")
```


---

> 📘 *This lesson is part of the [Python Concurrency: Asyncio & No-GIL](https://stanza.dev/courses/python-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*