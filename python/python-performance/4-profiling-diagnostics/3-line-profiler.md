---
source_course: "python-performance"
source_lesson: "python-performance-line-profiler"
---

# Line Profiler

For detailed analysis of specific functions.

## Installation

```bash
pip install line_profiler
```

## Usage

```python
# Add @profile decorator (no import needed)
@profile
def slow_function():
    result = []
    for i in range(1000):
        result.append(i ** 2)
    return sum(result)
```

```bash
# Run with kernprof
kernprof -l -v script.py
```

## Output Example

```
Line #      Hits         Time  Per Hit   % Time  Line Contents
==============================================================
     3                                           @profile
     4                                           def slow_function():
     5         1          2.0      2.0      0.0      result = []
     6      1001       1500.0      1.5     15.0      for i in range(1000):
     7      1000       8000.0      8.0     80.0          result.append(i ** 2)
     8         1        500.0    500.0      5.0      return sum(result)
```

## Programmatic Usage

```python
from line_profiler import LineProfiler

def target_function():
    # Code to profile
    pass

lp = LineProfiler()
lp.add_function(target_function)
lp.enable()

target_function()

lp.disable()
lp.print_stats()
```

## Code Examples

**Manual section timing**

```python
# Without line_profiler, use this approach
import time

def timed_section(name):
    """Context manager for timing code sections."""
    class Timer:
        def __enter__(self):
            self.start = time.perf_counter()
            return self
        def __exit__(self, *args):
            elapsed = time.perf_counter() - self.start
            print(f"{name}: {elapsed*1000:.2f}ms")
    return Timer()

# Usage
def process():
    with timed_section("data loading"):
        data = load_data()
    
    with timed_section("processing"):
        result = process_data(data)
    
    with timed_section("saving"):
        save(result)
```


---

> 📘 *This lesson is part of the [Python Performance: JIT & Internals](https://stanza.dev/courses/python-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*