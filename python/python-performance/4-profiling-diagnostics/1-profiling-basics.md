---
source_course: "python-performance"
source_lesson: "python-performance-profiling-basics"
---

# Why Profile?

> "Premature optimization is the root of all evil" - Donald Knuth

Before optimizing, measure to find actual bottlenecks.

## Types of Profilers

| Type | How it Works | Overhead | Accuracy |
|------|--------------|----------|----------|
| Deterministic | Hooks every call | High | Exact |
| Statistical | Samples periodically | Low | Approximate |
| Line-by-line | Per-line timing | Very High | Exact |

## Quick Timing

```python
import time

start = time.perf_counter()
result = expensive_operation()
elapsed = time.perf_counter() - start
print(f"Took {elapsed:.4f}s")
```

## timeit Module

```python
import timeit

# Time a statement
time_taken = timeit.timeit(
    'sum(range(1000))',
    number=10000
)
print(f"{time_taken:.4f}s for 10000 iterations")

# Time with setup
time_taken = timeit.timeit(
    'sorted(data)',
    setup='data = list(range(1000))',
    number=1000
)
```

## Command Line

```bash
# Quick timing
python -m timeit "sum(range(1000))"

# With setup
python -m timeit -s "data = list(range(1000))" "sorted(data)"
```

## Code Examples

**Simple benchmark function**

```python
import timeit

def benchmark(func, *args, iterations=1000):
    """Simple benchmarking function."""
    timer = timeit.Timer(lambda: func(*args))
    
    # Determine how many iterations for ~1 second
    times = timer.repeat(repeat=3, number=iterations)
    
    best = min(times) / iterations
    print(f"{func.__name__}: {best*1e6:.2f} µs per call")
    return best

# Usage
benchmark(sorted, list(range(1000)))
benchmark(lambda x: x.sort(), list(range(1000)))
```


---

> 📘 *This lesson is part of the [Python Performance: JIT & Internals](https://stanza.dev/courses/python-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*