---
source_course: "python-performance"
source_lesson: "python-performance-caching-memoization"
---

# Caching Strategies

## functools.lru_cache

```python
from functools import lru_cache

@lru_cache(maxsize=128)
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n-1) + fibonacci(n-2)

# Without cache: O(2^n)
# With cache: O(n)
print(fibonacci(100))  # Instant!

# Check cache stats
print(fibonacci.cache_info())
# CacheInfo(hits=98, misses=101, maxsize=128, currsize=101)
```

## @cache (Python 3.9+)

```python
from functools import cache

@cache  # Unbounded cache
def factorial(n):
    return n * factorial(n-1) if n else 1
```

## @cached_property (3.8+)

```python
from functools import cached_property

class Data:
    @cached_property
    def processed(self):
        # Only computed once
        return expensive_computation(self.raw)
```

## Custom Caching

```python
from functools import wraps
import time

def timed_cache(seconds):
    def decorator(func):
        cache = {}
        
        @wraps(func)
        def wrapper(*args):
            if args in cache:
                result, timestamp = cache[args]
                if time.time() - timestamp < seconds:
                    return result
            
            result = func(*args)
            cache[args] = (result, time.time())
            return result
        
        return wrapper
    return decorator

@timed_cache(60)  # Cache for 60 seconds
def fetch_data(url):
    return requests.get(url).json()
```

## Code Examples

**LRU cache in action**

```python
from functools import lru_cache
import time

@lru_cache(maxsize=1000)
def expensive_api_call(user_id):
    time.sleep(0.1)  # Simulate network delay
    return {"id": user_id, "name": f"User {user_id}"}

# First call: slow
start = time.time()
for i in range(10):
    expensive_api_call(i)
print(f"First run: {time.time() - start:.2f}s")

# Second call: instant (cached)
start = time.time()
for i in range(10):
    expensive_api_call(i)
print(f"Cached run: {time.time() - start:.4f}s")
```


---

> 📘 *This lesson is part of the [Python Performance: JIT & Internals](https://stanza.dev/courses/python-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*