---
source_course: "python-performance"
source_lesson: "python-performance-profiling-basics"
---

# Profiling Fundamentals

## Introduction

Optimizing code without profiling is guessing. Profilers reveal where your program actually spends its time, replacing assumptions with data. Understanding the different categories of profilers and when to use each is the foundation of systematic performance work.

## Key Concepts

- **Deterministic profiler**: Instruments every function call and return, providing exact call counts and timing but adding overhead that can distort results (e.g., cProfile).
- **Statistical (sampling) profiler**: Periodically samples the call stack without instrumenting every call, providing lower overhead at the cost of some precision (e.g., py-spy, scalene).
- **Line-by-line profiler**: Measures execution time for each line within a function, pinpointing exactly which statements are slow (e.g., line_profiler).
- **`timeit` module**: A micro-benchmarking tool that runs a code snippet many times and reports the best time, designed for comparing small pieces of code.

## Real World Context

A web application is slow, but you do not know whether the bottleneck is in database queries, JSON serialization, or template rendering. Running `cProfile` identifies which functions consume the most time. Once you find the expensive function, `line_profiler` reveals which exact lines within it to optimize. For production systems where you cannot afford instrumentation overhead, statistical profilers like `py-spy` attach to running processes without modifying code.

## Deep Dive

### Choosing the Right Profiler

Each profiler type answers a different question:

| Profiler Type | Question It Answers | Overhead | Precision |
|---|---|---|---|
| Deterministic (cProfile) | Which functions are slow? | Medium-High | Exact call counts |
| Statistical (py-spy) | Where does the program spend time? | Very Low | Approximate |
| Line-by-line (line_profiler) | Which lines in a function are slow? | High | Per-line timing |
| Micro-benchmark (timeit) | Which approach is faster? | None (isolated) | Nanosecond |

### Quick Timing with time.perf_counter

For ad-hoc measurements, `time.perf_counter()` provides the highest resolution monotonic clock:

```python
import time

start = time.perf_counter()
result = expensive_function()
elapsed = time.perf_counter() - start
print(f"Elapsed: {elapsed:.4f} seconds")
```

This is useful for sanity checks but not rigorous benchmarking — a single run is affected by system load, GC pauses, and JIT warmup.

### The timeit Module

`timeit` runs code many times and reports statistics, eliminating noise:

```python
import timeit

# Compare two approaches
time_join = timeit.timeit('"".join(words)', setup='words=["x"]*10000', number=1000)
time_plus = timeit.timeit(
    'result = ""\nfor w in words:\n    result += w',
    setup='words=["x"]*10000',
    number=1000
)
print(f"join: {time_join:.4f}s | +=: {time_plus:.4f}s")
```

From the command line:

```bash
python -m timeit -s "words=['x']*10000" "''.join(words)"
```

`timeit` automatically determines the number of iterations if you do not specify `number`, aiming for a total runtime of about 2 seconds.

### When to Use What

Start broad and narrow down:

1. **First**: Run `cProfile` to find which functions are slow (the 80/20 rule — 80% of time is usually in 20% of functions).
2. **Then**: Use `line_profiler` on the identified hot functions to find the slow lines.
3. **Finally**: Use `timeit` to compare alternative implementations of those lines.
4. **In production**: Use statistical profilers (py-spy, scalene) that attach without code changes.

## Common Pitfalls

- **Profiling optimized code with a high-overhead profiler**: Deterministic profilers add overhead per function call. If your code makes millions of tiny calls, the profiler overhead itself dominates the measurement, giving misleading results. Use statistical profilers for call-heavy code.
- **Benchmarking with `time.time()` instead of `time.perf_counter()`**: `time.time()` has lower resolution and can jump backwards during clock adjustments. Always use `perf_counter()` for timing code.
- **Optimizing without profiling first**: Developers often optimize code they think is slow rather than code that is actually slow. Always measure before optimizing.

## Best Practices

- Always profile before optimizing. Intuition about bottlenecks is unreliable.
- Use `timeit` for micro-benchmarks of isolated code snippets, not for profiling entire applications.
- Start with the lowest-overhead profiler that answers your question — use cProfile for function-level, py-spy for production, line_profiler for line-level.

## Summary

- Deterministic profilers (cProfile) instrument every call — exact but add overhead.
- Statistical profilers (py-spy, scalene) sample periodically — low overhead, ideal for production.
- Line-by-line profilers (line_profiler) pinpoint slow lines within a function.
- `timeit` is for comparing small code alternatives, not for profiling applications.
- The workflow is: cProfile to find hot functions, line_profiler to find hot lines, timeit to compare fixes.

## Code Examples

**Using timeit to rigorously compare list comprehension vs generator expression performance for summing squares**

```python
import timeit

# Compare list comprehension vs generator expression with sum()
setup = "data = list(range(100_000))"

listcomp = timeit.timeit("sum([x * x for x in data])", setup=setup, number=500)
genexpr = timeit.timeit("sum(x * x for x in data)", setup=setup, number=500)

print(f"List comprehension: {listcomp:.4f}s")
print(f"Generator expr:     {genexpr:.4f}s")
print(f"Ratio: {listcomp / genexpr:.2f}x")
```

**A context manager using perf_counter for clean ad-hoc timing of code blocks**

```python
import time

class Timer:
    """A reusable context manager for timing code blocks."""
    def __init__(self, label: str = "Block"):
        self.label = label
        self.elapsed = 0.0

    def __enter__(self):
        self.start = time.perf_counter()
        return self

    def __exit__(self, *args):
        self.elapsed = time.perf_counter() - self.start
        print(f"{self.label}: {self.elapsed:.4f}s")

# Usage
with Timer("Sorting"):
    sorted(range(1_000_000, 0, -1))

with Timer("List comprehension"):
    [x ** 2 for x in range(500_000)]
```


## Resources

- [timeit — Measure execution time of small code snippets](https://docs.python.org/3.14/library/timeit.html) — Official documentation for the timeit module including command-line and programmatic usage
- [time.perf_counter](https://docs.python.org/3.14/library/time.html#time.perf_counter) — Official documentation for the high-resolution performance counter

---

> 📘 *This lesson is part of the [Python Performance: JIT & Internals](https://stanza.dev/courses/python-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*