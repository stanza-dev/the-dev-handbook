---
source_course: "python-concurrency"
source_lesson: "python-concurrency-understanding"
---

# Concurrency vs Parallelism

These terms are often confused but represent different concepts.

## Concurrency

**Dealing with multiple tasks at once** - tasks can start, run, and complete in overlapping time periods, but not necessarily simultaneously.

```
Task A: ████----████----████
Task B: ----████----████----
Time:   →→→→→→→→→→→→→→→→→→→→
```

Concurrency is about **structure** - organizing code to handle multiple things.

## Parallelism

**Executing multiple tasks simultaneously** - actually running at the same time on multiple CPU cores.

```
Core 1: ████████████████████
Core 2: ████████████████████
Time:   →→→→→→→→→→→→→→→→→→→→
```

Parallelism is about **execution** - using hardware to do multiple things at once.

## Python's Concurrency Models

1. **Threading** - Concurrent, limited parallelism (GIL)
2. **Asyncio** - Concurrent, single-threaded
3. **Multiprocessing** - True parallelism
4. **Free-threading (3.13+)** - True parallelism with threads

## When to Use What

| Task Type | Best Approach |
|-----------|---------------|
| I/O-bound (network, files) | asyncio or threading |
| CPU-bound (computation) | multiprocessing or free-threading |
| Mixed | Combination approach |

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


---

> 📘 *This lesson is part of the [Python Concurrency: Asyncio & No-GIL](https://stanza.dev/courses/python-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*