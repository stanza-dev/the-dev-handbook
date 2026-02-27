---
source_course: "python-metaprogramming"
source_lesson: "python-metaprogramming-advanced-decorators"
---

# Parametrized Decorators

## Introduction

Sometimes a simple decorator is not enough -- you need to configure its behavior with arguments, like specifying how many times to retry a function or what permission level to require. Parametrized decorators solve this by adding an extra layer of nesting, turning the decorator itself into a factory that produces decorators on the fly.

## Key Concepts

- **Decorator factory** -- A function that accepts configuration arguments and returns a decorator. This is the outermost layer in the three-level nesting pattern.
- **Three-level nesting** -- The standard pattern for parametrized decorators: factory (takes config) -> decorator (takes function) -> wrapper (takes function args).
- **Optional arguments pattern** -- A technique that lets a decorator work both with and without parentheses, e.g., `@retry` and `@retry(times=5)`.

## Real World Context

Parametrized decorators are everywhere in professional Python codebases. Flask's `@app.route("/path", methods=["GET"])` is a decorator factory. Django's `@permission_required("can_edit")` uses this pattern. Rate limiters, circuit breakers, cache decorators with TTL, and feature flags all need configuration, making the three-level pattern a daily tool for backend developers.

## Deep Dive

A basic decorator only takes one argument: the function it decorates. But what if you want to pass configuration to the decorator? The solution is to add an outer function that captures the configuration and returns the actual decorator. This creates three levels of nesting:

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

Each level has a distinct responsibility: `repeat(n)` captures the configuration, `decorator(func)` captures the function, and `wrapper(*args, **kwargs)` runs when the decorated function is called.

It helps to trace through the execution step by step to see how Python resolves the `@` syntax:

```python
@repeat(3)
def say_hello(): ...

# Step 1: repeat(3) returns decorator
# Step 2: decorator(say_hello) returns wrapper
# Step 3: say_hello = wrapper
```

Python first evaluates `repeat(3)`, which returns the `decorator` function. Then it applies `decorator` to `say_hello`, which returns `wrapper`. Finally, `say_hello` is rebound to `wrapper`.

A more advanced technique lets a decorator work both with and without arguments. This is useful when you want sensible defaults but also allow customization. The trick is to make the first positional argument optional:

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

When used as `@retry` (no parentheses), Python passes the function directly as `func`, so the `if func is None` check fails and `decorator(func)` is called immediately. When used as `@retry(times=5)`, `func` is `None`, so the factory returns `decorator`, which Python then applies to the function below.

## Common Pitfalls

- **Mixing up the nesting levels.** It is easy to accidentally return the wrong function or close over the wrong variable. Remember: the outermost function takes config, the middle takes the function, and the innermost takes the function's arguments.
- **Forgetting parentheses with parametrized decorators.** Writing `@repeat` instead of `@repeat(3)` passes the decorated function as the `n` argument, causing a `TypeError` when `range(func)` is called. The optional arguments pattern avoids this but adds complexity.
- **Not using `functools.wraps` on the wrapper.** Even inside parametrized decorators, always apply `@functools.wraps(func)` to preserve the original function's metadata.

## Best Practices

- Use the three-level pattern whenever your decorator needs configuration. It is the standard, well-understood approach.
- Consider the optional arguments pattern only when you have sensible defaults and want the ergonomic benefit of supporting both `@retry` and `@retry(times=5)`.
- Always apply `@functools.wraps(func)` on the innermost wrapper function to preserve metadata through all layers of decoration.

## Summary

- Parametrized decorators use a three-level nesting pattern: factory -> decorator -> wrapper.
- The factory function captures configuration arguments and returns the actual decorator.
- Python evaluates `@repeat(3)` in two steps: first `repeat(3)` returns a decorator, then that decorator is applied to the function.
- The optional arguments pattern (`func=None, *, ...`) allows a decorator to work both with and without parentheses.
- Always use `@functools.wraps(func)` on the innermost wrapper, even in parametrized decorators.

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


## Resources

- [functools — Higher-order functions](https://docs.python.org/3.14/library/functools.html) — Official functools module documentation

---

> 📘 *This lesson is part of the [Python Metaprogramming & Introspection](https://stanza.dev/courses/python-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*