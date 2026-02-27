---
source_course: "python-concurrency"
source_lesson: "python-concurrency-understanding"
---

# Understanding Concurrency vs Parallelism

## Introduction
Concurrency and parallelism are two of the most commonly confused terms in programming, yet understanding the difference is essential for writing efficient Python code. If you have ever wondered why your multi-threaded program did not speed up a CPU-heavy task, this lesson will give you the mental model to choose the right approach every time.

## Key Concepts
- **Concurrency**: Dealing with multiple tasks at once. Tasks can start, run, and complete in overlapping time periods, but not necessarily simultaneously. Concurrency is about *structure*.
- **Parallelism**: Executing multiple tasks at the same time on multiple CPU cores. Parallelism is about *execution*.
- **I/O-bound work**: Tasks that spend most of their time waiting for external resources like network or disk.
- **CPU-bound work**: Tasks that spend most of their time performing computation.

## Real World Context
Imagine you are building a web scraper that fetches thousands of pages and then parses their HTML. The fetching part is I/O-bound (waiting for network responses), so concurrency with asyncio or threading gives huge speedups. The parsing part is CPU-bound, so true parallelism via multiprocessing or free-threading is the right tool. Choosing incorrectly means your program runs no faster than the serial version.

## Deep Dive

### Concurrency Visualized

```
Task A: ████----████----████
Task B: ----████----████----
Time:   →→→→→→→→→→→→→→→→→→→→
```

Concurrency is about organizing code to handle multiple things.

### Parallelism Visualized

```
Core 1: ████████████████████
Core 2: ████████████████████
Time:   →→→→→→→→→→→→→→→→→→→→
```

Parallelism is about using hardware to do multiple things at once.

### Python's Concurrency Models

1. **Threading** - Concurrent, limited parallelism (GIL)
2. **Asyncio** - Concurrent, single-threaded
3. **Multiprocessing** - True parallelism
4. **Free-threading (3.13+)** - True parallelism with threads

### When to Use What

| Task Type | Best Approach |
|-----------|---------------|
| I/O-bound (network, files) | asyncio or threading |
| CPU-bound (computation) | multiprocessing or free-threading |
| Mixed | Combination approach |

## Common Pitfalls
1. **Using threading for CPU-bound work** — Because of the GIL, threading will not speed up computation-heavy tasks in standard Python. Use multiprocessing or free-threading instead.
2. **Confusing concurrency with parallelism** — Concurrent code can run on a single core; it just interleaves tasks. Assuming concurrency gives you a multi-core speedup leads to disappointing benchmarks.

## Best Practices
1. **Profile before choosing** — Measure whether your bottleneck is I/O or CPU before selecting a concurrency model. The `time` module or `cProfile` can help.
2. **Start with asyncio for I/O-bound work** — It has lower overhead than threading and avoids many race-condition pitfalls since it runs on a single thread.

## Summary
- Concurrency is about structure (handling many tasks); parallelism is about execution (running tasks simultaneously).
- Python offers four main concurrency models: threading, asyncio, multiprocessing, and free-threading.
- I/O-bound tasks benefit from asyncio or threading; CPU-bound tasks need multiprocessing or free-threading.
- Always profile your workload before choosing a concurrency strategy.

## Code Examples

**Choosing the right approach**

```python
# I/O-bound: Network requests (asyncio is perfect)
import asyncio
import aiohttp

async def fetch_urls(urls):
    async with aiohttp.ClientSession() as session:
        tasks = [session.get(url) for url in urls]
        return await asyncio.gather(*tasks)

# CPU-bound: Number crunching (use multiprocessing)
from multiprocessing import Pool

def cpu_task(n):
    return sum(i**2 for i in range(n))

with Pool(4) as p:
    results = p.map(cpu_task, [10**6] * 4)
```


## Resources

- [Concurrent Execution](https://docs.python.org/3.14/library/concurrency.html) — Python standard library documentation for concurrent execution modules

---

> 📘 *This lesson is part of the [Python Concurrency: Asyncio & Free-Threading](https://stanza.dev/courses/python-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*