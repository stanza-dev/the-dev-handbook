---
source_course: "python-performance"
source_lesson: "python-performance-cprofile"
---

# Profiling with cProfile

## Introduction

cProfile is Python's built-in deterministic profiler. It traces every function call and return, recording how many times each function was called and how long each call took. It is the standard first tool for identifying performance bottlenecks in Python applications.

## Key Concepts

- **`ncalls`**: The number of times a function was called. A slash (e.g., `100/5`) means 100 total calls, 5 non-recursive.
- **`tottime`**: Total time spent inside the function itself, excluding time in subcalls. This answers "how much work does this function do directly?"
- **`cumtime`**: Cumulative time spent in the function including all subcalls. This answers "how much wall time does calling this function cost?"
- **`percall`**: Time per call — `tottime/ncalls` or `cumtime/ncalls` depending on the column.
- **`pstats`**: The standard library module for loading, filtering, and sorting cProfile output files.

## Real World Context

When a Django request takes 2 seconds but you do not know whether the database query, serialization, or middleware is the bottleneck, running cProfile on the request handler immediately reveals which functions consume the most time. In data pipelines, cProfile identifies whether data loading, transformation, or I/O is the dominant cost.

## Deep Dive

### Command Line Usage

The simplest way to profile a script:

```bash
python -m cProfile script.py
```

This prints all function calls sorted by the default order (filename/line). To sort by most useful metrics:

```bash
# Sort by total time in function (excluding subcalls)
python -m cProfile -s tottime script.py

# Sort by cumulative time (including subcalls)
python -m cProfile -s cumtime script.py

# Save to file for later analysis
python -m cProfile -o profile.prof script.py
```

### Understanding the Output

Here is a typical cProfile output:

```
         1000003 function calls in 0.842 seconds

   Ordered by: internal time

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
   1000000    0.563    0.000    0.563    0.000 example.py:10(process_item)
         1    0.205    0.205    0.842    0.842 example.py:5(main)
         1    0.074    0.074    0.637    0.637 example.py:15(process_all)
         1    0.000    0.000    0.842    0.842 {built-in method builtins.exec}
```

Reading this: `process_item` was called 1,000,000 times, spending 0.563 seconds total in the function itself. `main` spent 0.205 seconds in its own code but 0.842 seconds total (including subcalls).

### Programmatic Usage

For profiling specific sections of code:

```python
import cProfile
import pstats

profiler = cProfile.Profile()
profiler.enable()

# Code to profile
result = process_data(large_dataset)

profiler.disable()

# Print sorted stats
stats = pstats.Stats(profiler)
stats.sort_stats('tottime')
stats.print_stats(20)  # Top 20 functions
```

### Filtering with pstats

When the output is overwhelming, pstats lets you focus on what matters:

```python
import pstats

# Load saved profile
stats = pstats.Stats('profile.prof')

# Show only functions in your own code
stats.sort_stats('cumtime')
stats.print_stats('myproject/')  # Filter by path substring

# Show callers of a specific function
stats.print_callers('process_item')

# Show what a function calls
stats.print_callees('main')
```

### Using as a Context Manager

For clean scoping:

```python
import cProfile

with cProfile.Profile() as profiler:
    result = expensive_function()

profiler.print_stats('tottime')
```

## Common Pitfalls

- **Profiling too much code at once**: cProfile output for a full application run can have thousands of entries. Always narrow down to the specific request or operation you want to optimize.
- **Confusing tottime and cumtime**: If `cumtime` is high but `tottime` is low, the function itself is fast — it is spending time waiting on the functions it calls. Optimize those subcalls instead.
- **Not accounting for profiler overhead**: cProfile adds 10-30% overhead because it hooks into every function call and return. Tight loops with millions of tiny function calls will appear slower than they actually are.

## Best Practices

- Sort by `tottime` first to find functions doing the most direct work, then by `cumtime` to understand the call chain.
- Use `print_callers()` and `print_callees()` to trace the call graph when you find a hot function.
- Save profiles to files (`-o profile.prof`) so you can re-analyze without re-running the code.

## Summary

- `python -m cProfile -s tottime script.py` is the fastest way to find which functions consume the most direct CPU time.
- `tottime` measures time in the function only; `cumtime` includes all subcall time.
- Use `pstats.Stats` to filter, sort, and explore profile data programmatically.
- `print_callers()` and `print_callees()` trace the call graph around hot functions.
- cProfile adds measurable overhead — keep this in mind when interpreting results for call-heavy code.

## Code Examples

**Programmatically profiling a function with cProfile and printing the top results sorted by tottime**

```python
import cProfile
import pstats
import io

def slow_function():
    total = 0
    for i in range(500_000):
        total += i ** 2
    return total

def fast_function():
    return sum(i ** 2 for i in range(500_000))

def main():
    slow_function()
    fast_function()

# Profile and print top functions sorted by tottime
profiler = cProfile.Profile()
profiler.enable()
main()
profiler.disable()

stream = io.StringIO()
stats = pstats.Stats(profiler, stream=stream)
stats.sort_stats('tottime')
stats.print_stats(10)
print(stream.getvalue())
```

**Saving a cProfile to a file and using pstats to explore the call graph with print_callees**

```python
import cProfile
import pstats

# Profile and save to file
cProfile.run('import json; json.dumps(list(range(100_000)))', 'json_profile.prof')

# Analyze the saved profile
stats = pstats.Stats('json_profile.prof')
stats.sort_stats('cumtime')
stats.print_stats(10)

# Trace call relationships
print("\n--- Callees of dumps ---")
stats.print_callees('dumps')
```


## Resources

- [The Python Profilers (cProfile, profile, pstats)](https://docs.python.org/3.14/library/profile.html) — Official documentation for cProfile, the profile module, and pstats analysis tools

---

> 📘 *This lesson is part of the [Python Performance: JIT & Internals](https://stanza.dev/courses/python-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*