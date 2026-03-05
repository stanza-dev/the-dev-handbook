---
source_course: "python-performance"
source_lesson: "python-performance-memory-profiling"
---

# Memory Profiling

## Introduction

Knowing that Python objects have overhead is one thing; finding exactly where your application allocates the most memory is another. Python's built-in `tracemalloc` module lets you take memory snapshots, compare them, and track peak usage without any external dependencies. This lesson teaches you how to diagnose memory issues systematically.

## Key Concepts

- **`tracemalloc`**: Python's built-in memory allocation tracer that records where every allocation happens, including the exact file and line number.
- **Snapshot**: A frozen record of all current memory allocations at a point in time. You can compare two snapshots to see what changed.
- **Peak Memory**: The maximum amount of memory your program has used at any point during execution, tracked by `tracemalloc.get_traced_memory()`.
- **Traceback Depth**: The number of stack frames `tracemalloc` records for each allocation. Higher depth gives more context but uses more memory itself.
- **Memory Leak**: A situation where memory usage grows continuously because objects are unintentionally kept alive (e.g., appended to a list that is never cleared).

## Real World Context

A Django application that processes uploaded files might see its memory usage climb to several gigabytes over hours. Using `tracemalloc` snapshots before and after processing, you can identify that temporary buffers are being appended to a module-level list and never released.

## Deep Dive

### Getting Started with tracemalloc

The simplest way to find where memory is being allocated:

```python
import tracemalloc

tracemalloc.start()

# Your code that allocates memory
data = [i ** 2 for i in range(100_000)]
lookup = {str(i): i for i in range(50_000)}

snapshot = tracemalloc.take_snapshot()
top_stats = snapshot.statistics("lineno")

print("Top 5 memory allocations by line:")
for stat in top_stats[:5]:
    print(f"  {stat}")
```

This shows you which lines in your code are responsible for the most memory allocation.

### Comparing Snapshots to Find Leaks

The most powerful technique is comparing two snapshots to see what memory was allocated between them:

```python
import tracemalloc

tracemalloc.start()

# Take baseline snapshot
snapshot_before = tracemalloc.take_snapshot()

# Execute the code under investigation
results = process_large_dataset()

# Take snapshot after
snapshot_after = tracemalloc.take_snapshot()

# Compare: what's new?
diff_stats = snapshot_after.compare_to(snapshot_before, "lineno")

print("Memory changes (top 10):")
for stat in diff_stats[:10]:
    print(f"  {stat}")
```

Positive changes indicate new allocations; negative changes indicate freed memory. If you see large positive changes in unexpected places, you have found a potential memory leak.

### Tracking Peak Memory Usage

Monitor both current and peak memory during execution:

```python
import tracemalloc

tracemalloc.start()

# Run your workload
for batch in range(100):
    data = generate_batch(batch)
    process(data)
    del data  # Free batch memory

current, peak = tracemalloc.get_traced_memory()
print(f"Current memory: {current / 1024 / 1024:.1f} MB")
print(f"Peak memory:    {peak / 1024 / 1024:.1f} MB")

tracemalloc.stop()
```

If peak memory is much higher than current memory, your code is correctly freeing intermediate results. If they are close, memory is accumulating.

### Deep Tracebacks

Increase the traceback depth to see the full call chain that led to an allocation:

```python
import tracemalloc

# Store 25 stack frames per allocation (default is 1)
tracemalloc.start(25)

data = {i: str(i) * 100 for i in range(10_000)}

snapshot = tracemalloc.take_snapshot()
stats = snapshot.statistics("traceback")

# Show the largest allocation's full traceback
if stats:
    biggest = stats[0]
    print(f"{biggest.count} blocks: {biggest.size / 1024:.1f} KB")
    print("Traceback (most recent call last):")
    for line in biggest.traceback.format():
        print(f"  {line}")
```

### Filtering Snapshots

You can filter snapshots to focus on your code and exclude standard library allocations:

```python
import tracemalloc

tracemalloc.start()
# ... your code ...
snapshot = tracemalloc.take_snapshot()

# Exclude standard library and tracemalloc itself
filters = [
    tracemalloc.Filter(False, "<frozen importlib._bootstrap>"),
    tracemalloc.Filter(False, "<unknown>"),
    tracemalloc.Filter(False, tracemalloc.__file__),
]
filtered = snapshot.filter_traces(filters)

for stat in filtered.statistics("lineno")[:10]:
    print(stat)
```

### Practical Memory Profiling Pattern

Here is a reusable context manager for profiling any block of code:

```python
import tracemalloc
from contextlib import contextmanager

@contextmanager
def memory_profile(label=""):
    """Context manager that reports memory usage of a code block."""
    tracemalloc.start()
    snapshot_before = tracemalloc.take_snapshot()
    yield
    snapshot_after = tracemalloc.take_snapshot()
    current, peak = tracemalloc.get_traced_memory()
    tracemalloc.stop()

    print(f"\n--- Memory Profile{': ' + label if label else ''} ---")
    print(f"Peak: {peak / 1024 / 1024:.2f} MB")
    print(f"Current: {current / 1024 / 1024:.2f} MB")
    diff = snapshot_after.compare_to(snapshot_before, "lineno")
    for stat in diff[:5]:
        print(f"  {stat}")

# Usage
with memory_profile("data loading"):
    records = [{"id": i, "value": i * 3.14} for i in range(500_000)]
```

## Common Pitfalls

- **Forgetting to call `tracemalloc.stop()`**: tracemalloc itself uses memory to store allocation records. In production, always stop it after profiling to free this overhead.
- **Using tracemalloc in production with high frame depth**: `tracemalloc.start(25)` stores 25 stack frames per allocation, which uses significant memory. In production, use depth 1 or disable tracemalloc entirely.
- **Confusing `sys.getsizeof()` with `tracemalloc`**: `getsizeof()` measures the shallow size of one object. `tracemalloc` tracks all allocations made by the Python memory allocator. They measure different things.

## Best Practices

- **Compare snapshots to find leaks**: A single snapshot shows where memory is allocated. Comparing two snapshots over time reveals whether memory is growing unexpectedly.
- **Use the context manager pattern for ad-hoc profiling**: Wrap suspicious code blocks in a memory profiler to quickly identify which operations consume the most memory.
- **Filter out standard library noise**: Use `tracemalloc.Filter` to focus on your application code and ignore allocations from the standard library and import system.

## Summary

- `tracemalloc` is Python's built-in memory profiler that records allocation sites with file and line number information
- Comparing two snapshots (`snapshot_after.compare_to(snapshot_before)`) is the most effective way to find memory leaks
- `tracemalloc.get_traced_memory()` returns current and peak memory usage as a tuple
- Increase traceback depth with `tracemalloc.start(N)` to see the full call chain for large allocations, but be aware of the memory overhead
- Always stop tracemalloc after profiling to free its internal tracking data

## Code Examples

**Using tracemalloc to compare memory snapshots before and after creating a large data structure, identifying exactly which lines allocated the most memory**

```python
import tracemalloc

tracemalloc.start()

# Baseline snapshot
snap1 = tracemalloc.take_snapshot()

# Simulate work: create a large data structure
records = [
    {"id": i, "name": f"item_{i}", "value": i * 2.5}
    for i in range(100_000)
]

# Post-work snapshot
snap2 = tracemalloc.take_snapshot()

# Compare to find what was allocated
print("Top 5 memory increases:")
for stat in snap2.compare_to(snap1, "lineno")[:5]:
    print(f"  {stat}")

current, peak = tracemalloc.get_traced_memory()
print(f"\nCurrent: {current / 1024 / 1024:.1f} MB")
print(f"Peak:    {peak / 1024 / 1024:.1f} MB")

tracemalloc.stop()
```


## Resources

- [tracemalloc — Trace memory allocations](https://docs.python.org/3.14/library/tracemalloc.html) — Official documentation for tracemalloc including snapshots, filtering, and memory tracking
- [Memory Management in CPython](https://docs.python.org/3.14/c-api/memory.html) — CPython memory allocator internals and the memory management C API

---

> 📘 *This lesson is part of the [Python Performance: JIT & Internals](https://stanza.dev/courses/python-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*