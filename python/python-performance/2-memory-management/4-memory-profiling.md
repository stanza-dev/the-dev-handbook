---
source_course: "python-performance"
source_lesson: "python-performance-memory-profiling"
---

# Memory Profiling Tools

## tracemalloc (Built-in)

```python
import tracemalloc

tracemalloc.start()

# Your code here
data = [i ** 2 for i in range(100000)]

snapshot = tracemalloc.take_snapshot()
top_stats = snapshot.statistics('lineno')

print("Top 5 memory consumers:")
for stat in top_stats[:5]:
    print(stat)
```

## Comparing Snapshots

```python
import tracemalloc

tracemalloc.start()

# Baseline
snapshot1 = tracemalloc.take_snapshot()

# Do work
results = process_data()

# After
snapshot2 = tracemalloc.take_snapshot()

# Compare
top_stats = snapshot2.compare_to(snapshot1, 'lineno')
for stat in top_stats[:10]:
    print(stat)
```

## Memory Limit Tracking

```python
import tracemalloc

tracemalloc.start()

# ... code ...

current, peak = tracemalloc.get_traced_memory()
print(f"Current: {current / 1024 / 1024:.1f} MB")
print(f"Peak: {peak / 1024 / 1024:.1f} MB")

tracemalloc.stop()
```

## Third-Party Tools

```bash
# memory_profiler
pip install memory_profiler

# Usage
python -m memory_profiler script.py
```

```python
from memory_profiler import profile

@profile
def my_function():
    a = [1] * 1000000
    b = [2] * 2000000
    del b
    return a
```

## Code Examples

**Detailed memory tracing**

```python
import tracemalloc

tracemalloc.start(25)  # Store 25 stack frames

# Allocate memory
data = {i: str(i) * 100 for i in range(10000)}

snapshot = tracemalloc.take_snapshot()

# Group by traceback
stats = snapshot.statistics('traceback')

# Show the biggest allocation
biggest = stats[0]
print(f"{biggest.count} blocks: {biggest.size / 1024:.1f} KB")
for line in biggest.traceback.format():
    print(line)
```


---

> 📘 *This lesson is part of the [Python Performance: JIT & Internals](https://stanza.dev/courses/python-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*