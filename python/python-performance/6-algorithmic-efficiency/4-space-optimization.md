---
source_course: "python-performance"
source_lesson: "python-performance-space-optimization"
---

# Space-Time Tradeoffs

## Introduction

Every algorithm makes a tradeoff between time and space. You can often make code faster by using more memory (caching, lookup tables, precomputation) or reduce memory usage by accepting slower computation (generators, streaming, on-the-fly calculation). Understanding these tradeoffs lets you choose the right approach for your constraints.

## Key Concepts

- **Trading space for time**: Using additional memory (hash tables, caches, precomputed arrays) to avoid redundant computation and reduce time complexity.
- **Generators and lazy evaluation**: Python's `yield`-based generators produce values one at a time, processing data streams of any size with constant memory overhead.
- **Streaming processing**: Reading and processing data in chunks rather than loading everything into memory, essential for datasets larger than available RAM.
- **Memoization**: Caching function results to avoid recomputation, trading O(n) memory for O(1) lookup on repeated calls.

## Real World Context

Production systems constantly navigate space-time tradeoffs. A recommendation engine might precompute similarity scores for all product pairs (O(n^2) space) to serve instant recommendations (O(1) time), or compute them on-the-fly (O(1) space) at the cost of slower responses (O(n) time). The right choice depends on whether your constraint is memory (a 2GB container) or latency (a 100ms SLA).

## Deep Dive

### Trading Space for Time

The classic tradeoff: use more memory to run faster.

```python
# Example: Two-pass vs lookup table
# Task: Count character frequencies, then find first unique

# Approach 1: O(n) time, O(k) space (k = alphabet size)
def first_unique_char(s):
    count = {}  # Extra space: O(k)
    for c in s:
        count[c] = count.get(c, 0) + 1
    for c in s:
        if count[c] == 1:
            return c
    return None

# Approach 2: O(n^2) time, O(1) space
def first_unique_char_no_space(s):
    for i, c in enumerate(s):
        if s.index(c) == i and s.count(c) == 1:
            return c
    return None
```

### Generators for Memory Efficiency

Generators process data lazily, using constant memory:

```python
import sys

# MEMORY HUNGRY: List stores everything
def get_squares_list(n):
    return [i ** 2 for i in range(n)]  # All in memory

# MEMORY EFFICIENT: Generator yields one at a time
def get_squares_gen(n):
    for i in range(n):
        yield i ** 2  # One value at a time

# Compare memory usage
n = 1_000_000

squares_list = get_squares_list(n)
squares_gen = get_squares_gen(n)

print(f"List: {sys.getsizeof(squares_list):>10} bytes")  # ~8 MB
print(f"Generator: {sys.getsizeof(squares_gen):>6} bytes")  # ~200 bytes

# Both produce the same results
print(sum(get_squares_list(n)) == sum(get_squares_gen(n)))  # True
```

### Streaming vs Batch Processing

For files larger than memory:

```python
# BAD: Loads entire file into memory
def process_file_batch(filename):
    with open(filename) as f:
        data = f.readlines()  # All lines in memory!
    return [process_line(line) for line in data]

# GOOD: Streams line by line
def process_file_stream(filename):
    with open(filename) as f:
        for line in f:  # One line at a time
            yield process_line(line)

# GOOD: Process in chunks
def process_large_csv(filename, chunk_size=10_000):
    import csv
    with open(filename) as f:
        reader = csv.reader(f)
        chunk = []
        for row in reader:
            chunk.append(row)
            if len(chunk) >= chunk_size:
                yield process_chunk(chunk)
                chunk = []
        if chunk:
            yield process_chunk(chunk)
```

### Memoization: Trading Memory for Speed

```python
from functools import lru_cache

# Without memoization: O(2^n) time, O(n) space (call stack)
def fib_naive(n):
    if n < 2:
        return n
    return fib_naive(n - 1) + fib_naive(n - 2)

# With memoization: O(n) time, O(n) space (cache)
@lru_cache(maxsize=None)
def fib_memo(n):
    if n < 2:
        return n
    return fib_memo(n - 1) + fib_memo(n - 2)

# fib_naive(35) takes ~3 seconds
# fib_memo(35) takes ~0.00003 seconds
```

### Choosing the Right Approach

| Constraint | Strategy | Example |
|-----------|----------|--------|
| Memory-limited (containers, edge) | Generators, streaming | Process 100GB log files in 256MB container |
| Latency-limited (APIs, real-time) | Precompute, cache, lookup tables | Precompute user recommendations |
| Both limited | Streaming + bounded cache | LRU cache with maxsize |
| Neither limited | Whichever is simpler | Use readability as tiebreaker |

### itertools for Lazy Processing

```python
import itertools

# Chain multiple iterables without copying
combined = itertools.chain(range(1000), range(1000))

# Take first n items lazily
first_10 = itertools.islice(huge_generator(), 10)

# Group without loading all into memory
for key, group in itertools.groupby(sorted_data, key=keyfunc):
    process_group(key, group)

# Lazy filtering
positive = filter(lambda x: x > 0, huge_dataset)
```

## Common Pitfalls

1. **Unbounded caches** -- Using `@lru_cache` without `maxsize` (or `@cache`) on functions with many unique inputs can consume all available memory. Always set a reasonable maxsize or use `@lru_cache(maxsize=1024)`.
2. **Materializing generators accidentally** -- Calling `list()`, `len()`, or indexing on a generator loads everything into memory, defeating the purpose. Use `sum()`, `any()`, `all()`, or loop directly instead.
3. **Premature memory optimization** -- Using generators everywhere adds complexity. Only stream when data actually exceeds memory or when lazy evaluation provides a measurable benefit.

## Best Practices

1. **Default to generators for data pipelines** -- When processing data through multiple transformations, use generator expressions or `yield` to avoid creating intermediate lists.
2. **Set maxsize on all caches** -- Always use `@lru_cache(maxsize=N)` rather than `@cache` in production to prevent memory growth. Choose maxsize based on expected unique inputs.
3. **Match the strategy to the constraint** -- If memory is your bottleneck, stream with generators. If latency is your bottleneck, precompute and cache. If neither is constrained, choose the simpler approach.

## Summary

- Space-time tradeoffs are fundamental: you can trade memory for speed (caching) or speed for memory (streaming).
- Generators use constant memory regardless of data size, making them essential for large datasets.
- Streaming file processing (line by line or in chunks) handles files larger than available RAM.
- Memoization with `@lru_cache` converts exponential-time recursive algorithms to linear time.
- Choose your strategy based on whether memory or latency is the binding constraint.

## Code Examples

**Comparing memory usage between a list comprehension (O(n) memory) and a generator (O(1) memory) for the same computation, demonstrating constant-memory data processing**

```python
import sys

# Generators vs lists: memory comparison
def squares_list(n):
    """Returns a list: O(n) memory."""
    return [i ** 2 for i in range(n)]

def squares_generator(n):
    """Yields values: O(1) memory."""
    for i in range(n):
        yield i ** 2

n = 10_000_000

# List: allocates ~80 MB
big_list = squares_list(n)
print(f"List memory:      {sys.getsizeof(big_list) / 1024 / 1024:.1f} MB")

# Generator: allocates ~200 bytes
big_gen = squares_generator(n)
print(f"Generator memory: {sys.getsizeof(big_gen)} bytes")

# Both compute the same sum
del big_list  # Free memory first
print(f"\nGenerator sum: {sum(squares_generator(n))}")
# The generator never holds more than one value in memory!
```

**Demonstrating memoization with lru_cache: converting O(2^n) Fibonacci to O(n) by caching previously computed results, a classic space-time tradeoff**

```python
from functools import lru_cache
import time

# Memoization: trading O(n) space for exponential time savings
def fib_naive(n):
    """O(2^n) time -- recalculates same values millions of times."""
    if n < 2:
        return n
    return fib_naive(n - 1) + fib_naive(n - 2)

@lru_cache(maxsize=256)
def fib_cached(n):
    """O(n) time, O(n) space -- each value computed once."""
    if n < 2:
        return n
    return fib_cached(n - 1) + fib_cached(n - 2)

# fib_naive(35) takes seconds
start = time.perf_counter()
fib_naive(35)
naive_time = time.perf_counter() - start

# fib_cached(35) is instant
start = time.perf_counter()
fib_cached(35)
cached_time = time.perf_counter() - start

print(f"Naive:  {naive_time*1000:.0f}ms")
print(f"Cached: {cached_time*1000:.4f}ms")
print(f"Speedup: {naive_time/max(cached_time, 1e-9):.0f}x")
print(f"Cache stats: {fib_cached.cache_info()}")
```


## Resources

- [itertools -- Functions creating iterators for efficient looping](https://docs.python.org/3.14/library/itertools.html) — Standard library tools for memory-efficient iteration and lazy data processing
- [functools -- Higher-order functions and operations on callable objects](https://docs.python.org/3.14/library/functools.html) — Documentation for lru_cache, cache, and other function optimization tools

---

> 📘 *This lesson is part of the [Python Performance: JIT & Internals](https://stanza.dev/courses/python-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*