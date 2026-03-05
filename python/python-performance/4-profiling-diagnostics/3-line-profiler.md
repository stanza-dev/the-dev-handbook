---
source_course: "python-performance"
source_lesson: "python-performance-line-profiler"
---

# Line-by-Line Profiling

## Introduction

Once cProfile identifies a hot function, you need to know which specific lines within it are slow. The `line_profiler` package measures execution time for every line in decorated functions, giving you the precision needed to target optimizations at exactly the right code.

## Key Concepts

- **`line_profiler`**: A third-party package that hooks into Python's tracing mechanism to measure per-line execution time within decorated functions.
- **`kernprof`**: The command-line tool included with `line_profiler` that runs your script and collects per-line timing data.
- **`@profile` decorator**: A magic decorator injected by `kernprof` — you do not import it, just use it.
- **Output columns**: Hits (times executed), Time (total microseconds), Per Hit (microseconds per execution), % Time (percentage of function total).

## Real World Context

cProfile tells you that `parse_response()` takes 800ms, but the function is 40 lines long. Line profiling reveals that line 23 (a regex match) takes 600ms while everything else is fast. Now you know exactly what to optimize.

## Deep Dive

### Installation and Basic Usage

Install the package:

```bash
pip install line_profiler
```

Mark functions you want to profile with the `@profile` decorator (injected automatically by `kernprof`):

```python
# my_script.py
@profile
def process_data(items):
    results = []
    for item in items:
        cleaned = item.strip().lower()
        if cleaned not in seen:
            results.append(cleaned)
    return results

process_data(["  Hello ", "World  ", "  hello", "TEST"] * 10000)
```

Run with `kernprof`:

```bash
kernprof -l -v my_script.py
```

The `-l` flag enables line-by-line profiling and `-v` displays results immediately.

### Reading the Output

Typical output looks like this:

```
Line #      Hits         Time  Per Hit   % Time  Line Contents
==============================================================
     3                                           @profile
     4                                           def process_data(items):
     5         1          2.0      2.0      0.0      results = []
     6     40001      15234.0      0.4      8.2      for item in items:
     7     40000      58901.0      1.5     31.7      cleaned = item.strip().lower()
     8     40000      62453.0      1.6     33.6      if cleaned not in seen:
     9     10000      49234.0      4.9     26.5          results.append(cleaned)
    10         1          1.0      1.0      0.0      return results
```

The critical column is **% Time**. Here, `strip().lower()` and the membership check together account for 65% of execution time. These are your optimization targets.

### Programmatic Usage

You can use `line_profiler` without `kernprof` for integration into test suites or notebooks:

```python
from line_profiler import LineProfiler

def target_function(data):
    result = []
    for item in data:
        result.append(item ** 2)
    return sum(result)

profiler = LineProfiler()
profiler.add_function(target_function)

wrapped = profiler(target_function)
wrapped(range(100_000))

profiler.print_stats()
```

This is especially useful when you want to profile functions you cannot easily decorate in source code, or when running in Jupyter notebooks.

### Profiling Multiple Functions

You can profile several related functions in one run:

```python
from line_profiler import LineProfiler

profiler = LineProfiler()
profiler.add_function(parse_header)
profiler.add_function(parse_body)
profiler.add_function(validate)

wrapped_main = profiler(process_request)
wrapped_main(request_data)

profiler.print_stats()
```

All three functions will appear in the output if they were called during execution.

## Common Pitfalls

- **Leaving `@profile` in production code**: The `@profile` decorator is injected by `kernprof` at runtime. If you leave it in source code and run without `kernprof`, you get `NameError: name 'profile' is not defined`. Remove decorators before committing or add a dummy: `try: profile except NameError: profile = lambda f: f`.
- **Profiling functions that are too short**: Line profiler overhead per line execution is measurable. Functions called millions of times with 2-3 lines will show distorted results. Profile the caller instead.
- **Ignoring the 'Hits' column**: A line with low Per Hit but high Hits may dominate total time. Always check both columns.

## Best Practices

- Use cProfile first to identify hot functions, then apply line_profiler only to those functions.
- Focus optimization effort on lines with the highest % Time — even a 50% improvement on a 1% line yields only 0.5% total improvement.
- Use programmatic `LineProfiler` in Jupyter notebooks and test suites for reproducible profiling workflows.

## Summary

- `line_profiler` with `kernprof -l -v script.py` provides per-line execution time within decorated functions.
- The output shows Hits, Time, Per Hit, and % Time for each line — focus on % Time to identify optimization targets.
- Use the programmatic `LineProfiler` API for integration into notebooks and test frameworks.
- Always combine with cProfile: first find hot functions (cProfile), then find hot lines (line_profiler).

## Code Examples

**Using LineProfiler programmatically to profile a statistics function and identify which lines (sum, sorted, variance) dominate**

```python
from line_profiler import LineProfiler

def compute_stats(data):
    """Compute basic statistics on a list of numbers."""
    n = len(data)
    mean = sum(data) / n
    variance = sum((x - mean) ** 2 for x in data) / n
    std_dev = variance ** 0.5
    sorted_data = sorted(data)
    median = sorted_data[n // 2]
    return {'mean': mean, 'std': std_dev, 'median': median}

# Profile it
profiler = LineProfiler()
profiler.add_function(compute_stats)
wrapped = profiler(compute_stats)

import random
data = [random.gauss(0, 1) for _ in range(500_000)]
wrapped(data)

profiler.print_stats()
```


## Resources

- [line_profiler and kernprof](https://docs.python.org/3.14/library/profile.html) — Python profiling documentation covering the broader profiling ecosystem

---

> 📘 *This lesson is part of the [Python Performance: JIT & Internals](https://stanza.dev/courses/python-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*