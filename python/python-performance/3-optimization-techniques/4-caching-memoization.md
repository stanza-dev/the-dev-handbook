---
source_course: "python-performance"
source_lesson: "python-performance-caching-memoization"
---

# Caching & Memoization

## Introduction

Memoization transforms expensive repeated computations into dictionary lookups by caching results of previous calls. Python's `functools` module provides battle-tested decorators that handle cache management, eviction, and thread safety out of the box.

## Key Concepts

- **`@functools.lru_cache(maxsize=N)`**: A bounded Least Recently Used cache that evicts the oldest entry when full. Arguments must be hashable.
- **`@functools.cache`**: An unbounded cache (equivalent to `lru_cache(maxsize=None)`) that grows without limit. Introduced in Python 3.9.
- **`@functools.cached_property`**: Caches the result of a property method on the instance, computed once on first access.
- **Cache key**: The tuple of all arguments to the function. Unhashable arguments (lists, dicts) cannot be used as cache keys.

## Real World Context

Recursive algorithms (Fibonacci, dynamic programming), database query deduplication, API response caching, and configuration property computation are all common uses for memoization. The choice between bounded and unbounded caches depends on whether the argument space is finite and predictable.

## Deep Dive

### lru_cache: Bounded Memoization

`lru_cache` wraps a function with a dictionary-based cache that tracks access order:

```python
from functools import lru_cache

@lru_cache(maxsize=128)
def fibonacci(n: int) -> int:
    if n < 2:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)

# Without cache: O(2^n) — exponential
# With cache: O(n) — each value computed once
print(fibonacci(100))  # Instant
```

You can inspect cache performance at runtime:

```python
print(fibonacci.cache_info())
# CacheInfo(hits=98, misses=101, maxsize=128, currsize=101)

# Clear the cache when needed
fibonacci.cache_clear()
```

### @cache: Unbounded Memoization

Python 3.9 introduced `@cache` as shorthand for `lru_cache(maxsize=None)`:

```python
from functools import cache

@cache
def factorial(n: int) -> int:
    return n * factorial(n - 1) if n else 1

print(factorial(100))  # Computed once, then cached forever
```

The key risk is that unbounded caches grow without limit. If the function is called with many unique arguments (e.g., user IDs, timestamps), memory usage grows indefinitely.

### @cached_property: One-Time Computation

`cached_property` replaces itself with the computed value on first access:

```python
from functools import cached_property

class Dataset:
    def __init__(self, path: str):
        self.path = path

    @cached_property
    def stats(self) -> dict:
        """Expensive: reads entire file, computed only once."""
        data = open(self.path).read()
        return {
            'lines': data.count('\n'),
            'chars': len(data),
            'words': len(data.split())
        }

ds = Dataset('large_file.txt')
print(ds.stats)  # Computed on first access
print(ds.stats)  # Returns cached value instantly
```

To invalidate, delete the attribute: `del ds.stats`. The next access recomputes it.

### Custom Timed Cache

Sometimes you need cache entries to expire after a time window:

```python
from functools import wraps
import time

def timed_cache(seconds: float):
    """Cache results for a fixed duration."""
    def decorator(func):
        cache = {}

        @wraps(func)
        def wrapper(*args):
            now = time.monotonic()
            if args in cache:
                result, timestamp = cache[args]
                if now - timestamp < seconds:
                    return result
            result = func(*args)
            cache[args] = (result, now)
            return result

        wrapper.cache_clear = cache.clear
        return wrapper
    return decorator

@timed_cache(seconds=60)
def fetch_config(key: str) -> str:
    """Re-fetches from database after 60 seconds."""
    return db.query(f"SELECT value FROM config WHERE key = '{key}'")
```

## Common Pitfalls

- **Using `@cache` (unbounded) with a large or infinite argument space**: Every unique argument tuple is stored forever. For functions called with user IDs, request data, or timestamps, memory grows without bound — use `lru_cache(maxsize=N)` instead.
- **Caching functions with mutable or unhashable arguments**: `lru_cache` requires all arguments to be hashable. Passing a list or dict raises `TypeError`. Convert to tuple or frozenset before passing.
- **Forgetting that `cached_property` requires a mutable instance dict**: If the class uses `__slots__` without `__dict__`, `cached_property` cannot store its value and will recompute on every access.

## Best Practices

- Use `lru_cache(maxsize=N)` with a finite maxsize for functions where the argument space is large or unpredictable.
- Monitor cache performance with `func.cache_info()` during development to tune `maxsize` and confirm the cache is effective.
- Prefer `@cached_property` over manual caching for expensive instance attributes that do not change after initialization.

## Summary

- `@lru_cache(maxsize=N)` provides bounded memoization with LRU eviction, turning O(2^n) recursive algorithms into O(n).
- `@cache` is unbounded — convenient but dangerous when the argument space is large, as it can cause memory leaks.
- `@cached_property` computes a property value once on first access and stores it on the instance.
- Custom cache decorators can add time-based expiration for use cases like API or config caching.
- Always inspect `cache_info()` to verify hit rates and tune cache sizing.

## Code Examples

**Demonstrating lru_cache hit/miss behavior and measuring the speedup between first and cached calls**

```python
from functools import lru_cache
import time

@lru_cache(maxsize=256)
def expensive_computation(n: int) -> int:
    """Simulate expensive work."""
    time.sleep(0.01)  # 10ms per call
    return n ** 3 + n ** 2 + n

# First pass: all misses (slow)
start = time.perf_counter()
results = [expensive_computation(i) for i in range(100)]
first_pass = time.perf_counter() - start

# Second pass: all hits (instant)
start = time.perf_counter()
results = [expensive_computation(i) for i in range(100)]
second_pass = time.perf_counter() - start

print(f"First pass:  {first_pass:.3f}s")
print(f"Second pass: {second_pass:.6f}s")
print(f"Speedup:     {first_pass / second_pass:.0f}x")
print(f"Cache info:  {expensive_computation.cache_info()}")
```

**A custom time-based cache decorator that expires entries after a configurable number of seconds**

```python
from functools import wraps
import time

def timed_cache(seconds: float):
    """Decorator that caches results for a given number of seconds."""
    def decorator(func):
        cache = {}

        @wraps(func)
        def wrapper(*args):
            now = time.monotonic()
            if args in cache:
                result, ts = cache[args]
                if now - ts < seconds:
                    return result
            result = func(*args)
            cache[args] = (result, now)
            return result

        wrapper.cache = cache
        wrapper.cache_clear = cache.clear
        return wrapper
    return decorator

@timed_cache(seconds=2.0)
def get_data(key: str) -> str:
    print(f"  Computing for {key}...")
    return f"result-{key}-{time.time():.0f}"

print(get_data("a"))  # Computes
print(get_data("a"))  # Cached
time.sleep(2.5)
print(get_data("a"))  # Expired, recomputes
```


## Resources

- [functools.lru_cache](https://docs.python.org/3.14/library/functools.html#functools.lru_cache) — Official documentation for lru_cache including maxsize, typed parameter, and cache_info()
- [functools.cached_property](https://docs.python.org/3.14/library/functools.html#functools.cached_property) — Official documentation for cached_property and its interaction with __slots__

---

> 📘 *This lesson is part of the [Python Performance: JIT & Internals](https://stanza.dev/courses/python-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*