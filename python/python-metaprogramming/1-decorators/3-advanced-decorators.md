---
source_course: "python-metaprogramming"
source_lesson: "python-metaprogramming-advanced-decorators"
---

# Decorators with Arguments

## The Three-Level Pattern

To pass arguments to a decorator, add another layer:

```python
def repeat(n):           # Level 1: Takes decorator args
    def decorator(func):  # Level 2: Takes function
        @functools.wraps(func)
        def wrapper(*args, **kwargs):  # Level 3: Takes function args
            for _ in range(n):
                result = func(*args, **kwargs)
            return result
        return wrapper
    return decorator

@repeat(3)
def say_hello():
    print("Hello!")

say_hello()  # Prints "Hello!" three times
```

## How It Works

```python
@repeat(3)
def say_hello(): ...

# Step 1: repeat(3) returns decorator
# Step 2: decorator(say_hello) returns wrapper
# Step 3: say_hello = wrapper
```

## Optional Arguments Pattern

```python
def retry(func=None, *, times=3, delay=1):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(times):
                try:
                    return func(*args, **kwargs)
                except Exception:
                    if attempt < times - 1:
                        time.sleep(delay)
            raise
        return wrapper
    
    if func is None:
        return decorator  # Called with arguments
    return decorator(func)  # Called without arguments

# Both work:
@retry
def func1(): ...

@retry(times=5, delay=2)
def func2(): ...
```

## Code Examples

**Rate limiting decorator**

```python
import functools
import time

def rate_limit(calls_per_second):
    """Decorator to limit how often a function can be called."""
    min_interval = 1.0 / calls_per_second
    last_call = [0.0]  # Mutable to persist across calls
    
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            elapsed = time.time() - last_call[0]
            if elapsed < min_interval:
                time.sleep(min_interval - elapsed)
            last_call[0] = time.time()
            return func(*args, **kwargs)
        return wrapper
    return decorator

@rate_limit(2)  # Max 2 calls per second
def api_call():
    print(f"Called at {time.time():.2f}")
```


---

> 📘 *This lesson is part of the [Python Metaprogramming & Introspection](https://stanza.dev/courses/python-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*