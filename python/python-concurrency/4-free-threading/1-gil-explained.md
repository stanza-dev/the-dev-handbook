---
source_course: "python-concurrency"
source_lesson: "python-concurrency-no-gil-explained"
---

# Understanding Free-Threading

## Introduction
Free-threaded Python (PEP 703, PEP 779) is the most significant change to CPython's execution model in decades: it removes the Global Interpreter Lock, allowing multiple OS threads to execute Python bytecode truly in parallel. This lesson covers what free-threading is, how it evolved, its performance characteristics, and how to check whether your Python build supports it.

## Key Concepts
- **Free-threading**: A CPython build mode that removes the GIL, enabling true multi-threaded parallelism for Python code.
- **PEP 703**: The proposal to make the GIL optional in CPython, first implemented experimentally in Python 3.13.
- **PEP 779**: The acceptance of free-threading as officially supported in Python 3.14.
- **'t' suffix**: Free-threaded Python builds are identified by a 't' in their version string (e.g., `3.13t`).

## Real World Context
A scientific computing team runs CPU-heavy simulations in Python. Previously, they had to use multiprocessing (with its serialization overhead and memory duplication) or rewrite hot loops in C. With free-threaded Python, they can use plain `threading.Thread` to distribute computation across all cores, getting near-linear speedup without changing their data structures or serializing arguments.

## Deep Dive

### The Evolution

- **Python 3.13**: Experimental free-threading support (PEP 703)
- **Python 3.14**: Officially supported (PEP 779), performance penalty reduced to 5-10% on single-threaded code

### Performance Characteristics

| Scenario | GIL Python | Free-Threaded |
|----------|------------|---------------|
| Single-threaded | Baseline | ~5-10% slower (improved in 3.14) |
| Multi-threaded CPU | No speedup | Near-linear scaling |
| Multi-threaded I/O | Good | Similar |

### Checking Your Build

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

### Installing Free-Threaded Python

```bash
# Using pyenv
pyenv install 3.13t  # 't' suffix for free-threaded

# Using conda
conda create -n freethreaded python=3.13 python_abi=cp313t
```

## Common Pitfalls
1. **Assuming free-threading is always faster** — Single-threaded code runs 5-10% slower in free-threaded builds due to the overhead of fine-grained locking. Only use it when your workload benefits from true parallelism.
2. **Expecting all libraries to work immediately** — Many C extensions and third-party libraries were written assuming the GIL. They may crash or produce incorrect results until they are updated for free-threading.

## Best Practices
1. **Test with both GIL and free-threaded builds** — Run your test suite on both builds to catch thread-safety issues that were hidden by the GIL.
2. **Use `sys._is_gil_enabled()` for conditional behavior** — When writing library code that must work on both builds, check the GIL status at runtime to choose the appropriate strategy.

## Summary
- Free-threaded Python removes the GIL, enabling true multi-threaded parallelism for CPU-bound Python code.
- It was introduced experimentally in Python 3.13 (PEP 703) and became officially supported in Python 3.14 (PEP 779).
- Single-threaded performance is 5-10% slower, but multi-threaded CPU-bound work scales near-linearly.
- Use `sys._is_gil_enabled()` to check your build and install free-threaded builds with the 't' suffix.
- Not all libraries are compatible yet; test thoroughly before adopting.

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


## Resources

- [PEP 703 — Making the GIL Optional](https://peps.python.org/pep-0703/) — The proposal to make the Global Interpreter Lock optional in CPython

---

> 📘 *This lesson is part of the [Python Concurrency: Asyncio & Free-Threading](https://stanza.dev/courses/python-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*