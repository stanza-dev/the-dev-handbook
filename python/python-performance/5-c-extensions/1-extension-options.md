---
source_course: "python-performance"
source_lesson: "python-performance-extension-options"
---

# Extension Options Overview

## Introduction

When pure Python isn't fast enough, you have several options for dropping into native code. The key is knowing when each option is appropriate and starting with the simplest solution that meets your performance requirements. Python 3.14 also introduces `concurrent.interpreters` (PEP 734) as a new avenue for true multi-core parallelism without leaving Python.

## Key Concepts

- **C Extension API**: The official CPython interface for writing Python modules in C, offering maximum control but requiring manual reference counting and memory management.
- **Cython**: A superset of Python that compiles to C, providing near-C performance with Python-like syntax and automatic reference counting.
- **PyO3 (Rust)**: A Rust crate for building Python extensions with memory safety guarantees and zero-cost abstractions.
- **cffi / ctypes**: Foreign function interfaces for calling existing C libraries without writing C extension code.
- **concurrent.interpreters**: New in Python 3.14, this module (PEP 734) enables true multi-core parallelism using sub-interpreters, each with their own GIL.

## Real World Context

Most high-performance Python libraries use native extensions under the hood. NumPy's array operations are written in C, pydantic v2 rewrote its core in Rust (via PyO3), and polars is a Rust-based DataFrame library that consistently outperforms pandas. Before reaching for extensions, however, many performance problems can be solved with built-in C-speed functions like `sum()`, `sorted()`, and `map()`.

## Deep Dive

### The Decision Tree

Before writing native code, follow this decision tree:

1. **Can PyPy run your code?** PyPy's JIT can give 5-10x speedups with zero code changes.
2. **Can NumPy/pandas vectorize it?** Array operations in NumPy run at C speed.
3. **Did you profile first?** Use cProfile to confirm the bottleneck is CPU-bound.
4. **Is the bottleneck in a tight loop?** Extensions shine for CPU-bound inner loops.
5. **Do you need to call an existing C library?** Use ctypes or cffi.
6. **Need to write new native code?** Choose Cython or PyO3.

### Comparison Table

| Technology | Language | Learning Curve | Performance | Safety | Build Complexity |
|-----------|----------|---------------|-------------|--------|------------------|
| C Extension API | C | Steep | Maximum | Manual memory | Medium |
| Cython | Python-like | Moderate | Near-C | Semi-automatic | Medium |
| PyO3 | Rust | Steep | Near-C | Memory-safe | Easy (maturin) |
| cffi | Python | Low | Good | Automatic | Low |
| ctypes | Python | Low | Good | Manual types | None |

### Quick Wins With Built-ins First

Before reaching for extensions, try Python's built-in C-speed functions:

```python
# These are implemented in C and very fast
total = sum(numbers)           # C loop
sorted_items = sorted(data)    # Timsort in C
result = list(map(func, data)) # C iteration
any_match = any(x > 0 for x in data)  # Short-circuit in C
```

### Python 3.14: concurrent.interpreters

Python 3.14 introduces `concurrent.interpreters` (PEP 734), enabling true multi-core parallelism through sub-interpreters:

```python
from concurrent.futures import InterpreterPoolExecutor

def cpu_bound_task(n):
    return sum(i * i for i in range(n))

# Each interpreter has its own GIL - true parallelism!
with InterpreterPoolExecutor(max_workers=4) as executor:
    futures = [executor.submit(cpu_bound_task, 10_000_000)
               for _ in range(4)]
    results = [f.result() for f in futures]
```

This provides a new middle ground: multi-core performance without writing C or Rust, though with restrictions on shared mutable state between interpreters.

## Common Pitfalls

1. **Premature extension writing** -- Writing a C extension before profiling often means optimizing code that isn't the actual bottleneck, wasting days of development time on marginal gains.
2. **Ignoring the GIL context** -- C extensions that release the GIL enable true parallelism, but extensions that hold the GIL provide no threading benefit. Always consider whether your extension needs `Py_BEGIN_ALLOW_THREADS`.
3. **Choosing complexity over simplicity** -- Reaching for the C API when ctypes or Cython would suffice adds maintenance burden and increases the risk of memory leaks and segfaults.

## Best Practices

1. **Profile first, extend second** -- Always confirm the bottleneck with cProfile before investing in native code. The most impactful optimization is often algorithmic, not linguistic.
2. **Start with the simplest option** -- Try built-in functions, then NumPy, then Cython, then Rust/C. Each step adds complexity that must be justified by measurable gains.
3. **Consider concurrent.interpreters for CPU-bound work** -- In Python 3.14, `InterpreterPoolExecutor` offers true multi-core parallelism without leaving Python, making it the first thing to try before native extensions for parallelizable workloads.

## Summary

- Follow the decision tree: PyPy, then NumPy, then profile, then extensions.
- Built-in functions like `sum()`, `sorted()`, and `map()` are implemented in C and often fast enough.
- Python 3.14's `concurrent.interpreters` provides true multi-core parallelism without native code.
- Cython offers the best balance of ease-of-use and performance for most extension needs.
- Reserve the C API and Rust (PyO3) for maximum performance or when memory safety is critical.

## Code Examples

**Demonstrating that built-in C-speed functions like sum() can be 5-10x faster than manual Python loops, often eliminating the need for native extensions**

```python
# Decision tree in practice: try built-ins first
import time

numbers = list(range(1_000_000))

# Slow: manual Python loop
start = time.perf_counter()
total = 0
for n in numbers:
    total += n
loop_time = time.perf_counter() - start

# Fast: built-in sum() is implemented in C
start = time.perf_counter()
total = sum(numbers)
builtin_time = time.perf_counter() - start

print(f"Python loop: {loop_time*1000:.2f}ms")
print(f"Built-in sum: {builtin_time*1000:.2f}ms")
print(f"Speedup: {loop_time/builtin_time:.1f}x")
```

**Using Python 3.14's InterpreterPoolExecutor for true multi-core parallelism without writing any C or Rust code**

```python
from concurrent.futures import InterpreterPoolExecutor
import time

def cpu_work(n):
    """CPU-bound task: sum of squares."""
    return sum(i * i for i in range(n))

N = 5_000_000
TASKS = 4

# Sequential execution
start = time.perf_counter()
results_seq = [cpu_work(N) for _ in range(TASKS)]
seq_time = time.perf_counter() - start

# Parallel with InterpreterPoolExecutor (Python 3.14)
start = time.perf_counter()
with InterpreterPoolExecutor(max_workers=4) as pool:
    results_par = list(pool.map(cpu_work, [N] * TASKS))
par_time = time.perf_counter() - start

print(f"Sequential: {seq_time:.2f}s")
print(f"Parallel:   {par_time:.2f}s")
print(f"Speedup:    {seq_time/par_time:.1f}x")
```


## Resources

- [Extending Python with C or C++](https://docs.python.org/3.14/extending/extending.html) — Official guide to writing C extensions for CPython
- [concurrent.interpreters module](https://docs.python.org/3.14/library/concurrent.interpreters.html) — Python 3.14 sub-interpreters for multi-core parallelism (PEP 734)

---

> 📘 *This lesson is part of the [Python Performance: JIT & Internals](https://stanza.dev/courses/python-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*