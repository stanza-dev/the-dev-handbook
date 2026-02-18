---
source_course: "django-performance"
source_lesson: "django-performance-memory-profiling"
---

# Memory Profiling

## Introduction

Memory leaks and excessive memory usage can crash your application. Profiling helps identify memory issues.

## Key Concepts

**Memory Leak**: Objects not garbage collected.

**Memory Profile**: Snapshot of memory usage.

## Deep Dive

### Using memory_profiler

```python
# pip install memory_profiler
from memory_profiler import profile

@profile
def memory_heavy_function():
    data = []
    for i in range(1000000):
        data.append({'id': i, 'value': i * 2})
    return len(data)

# Output shows line-by-line memory usage:
# Line    Mem usage    Increment
# 3       50.0 MiB     0.0 MiB
# 5      150.0 MiB   100.0 MiB
```

### Finding Memory Leaks

```python
import tracemalloc

tracemalloc.start()

# Your code here
process_large_dataset()

snapshot = tracemalloc.take_snapshot()
top_stats = snapshot.statistics('lineno')

print('Top 10 memory allocations:')
for stat in top_stats[:10]:
    print(stat)
```

### Optimizing QuerySet Memory

```python
# BAD: Loads all objects into memory
for article in Article.objects.all():
    process(article)

# GOOD: Process in chunks
from django.core.paginator import Paginator

paginator = Paginator(Article.objects.all(), 1000)
for page_num in paginator.page_range:
    for article in paginator.page(page_num):
        process(article)

# BETTER: Use iterator()
for article in Article.objects.iterator(chunk_size=1000):
    process(article)
```

## Best Practices

1. **Use iterator() for large querysets**: Doesn't cache results.
2. **Process in batches**: Prevents memory spikes.
3. **Monitor in production**: Use APM tools like New Relic.

## Summary

Use memory_profiler to identify memory-heavy code. Use iterator() and chunking for large querysets. Monitor memory in production with APM tools.

## Resources

- [Memory Profiler](https://pypi.org/project/memory-profiler/) — Python memory profiler

---

> 📘 *This lesson is part of the [Django Performance & Optimization](https://stanza.dev/courses/django-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*