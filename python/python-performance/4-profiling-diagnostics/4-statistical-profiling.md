---
source_course: "python-performance"
source_lesson: "python-performance-statistical-profiling"
---

# Statistical Profilers

Lower overhead than deterministic profilers.

## py-spy

```bash
pip install py-spy

# Profile running process
py-spy top --pid 12345

# Record flame graph
py-spy record -o profile.svg -- python script.py
```

## scalene

```bash
pip install scalene

# Run with profiling
scalene script.py

# Profile specific functions
scalene --profile-only mymodule script.py
```

Scalene provides:
- CPU time (Python vs C)
- Memory allocation
- GPU usage
- Line-by-line breakdown

## Profiling Async Code

```python
import asyncio
import cProfile

async def main():
    await asyncio.gather(
        task1(),
        task2()
    )

# Profile the event loop
with cProfile.Profile() as pr:
    asyncio.run(main())

# Note: cProfile doesn't show async context well
# Use yappi for better async support
```

## yappi (Yet Another Python Profiler)

```python
import yappi
import asyncio

async def main():
    await work()

yappi.set_clock_type("wall")  # or "cpu"
yappi.start()

asyncio.run(main())

yappi.stop()
stats = yappi.get_func_stats()
stats.print_all()
```

## Code Examples

**Profiling context manager**

```python
# Profiling context for quick tests
import cProfile
import pstats
from contextlib import contextmanager

@contextmanager
def profiling(sort_by='cumulative', limit=20):
    """Context manager for quick profiling."""
    pr = cProfile.Profile()
    pr.enable()
    try:
        yield pr
    finally:
        pr.disable()
        stats = pstats.Stats(pr)
        stats.sort_stats(sort_by)
        stats.print_stats(limit)

# Usage
with profiling():
    result = expensive_operation()
```


---

> 📘 *This lesson is part of the [Python Performance: JIT & Internals](https://stanza.dev/courses/python-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*