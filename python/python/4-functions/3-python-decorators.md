---
source_course: "python"
source_lesson: "python-decorators"
---

# Decorators

## Introduction
Decorators let you add behavior to functions or methods without modifying their source code. They are one of Python's most elegant patterns and power everything from Flask routes to pytest fixtures. This lesson walks you through writing, stacking, and parameterizing decorators.

## Key Concepts
- **Decorator**: A function that takes another function and returns an enhanced version of it.
- **`@decorator` syntax**: Syntactic sugar for `func = decorator(func)`.
- **`functools.wraps`**: Preserves the original function's name, docstring, and metadata on the wrapper.
- **Parameterized decorator**: A decorator factory that accepts arguments and returns the actual decorator.

## Real World Context
Decorators are everywhere in production Python: `@app.route` in Flask, `@login_required` in Django, `@retry` in tenacity, `@lru_cache` in functools. They enable cross-cutting concerns like logging, timing, authentication, and caching to be applied declaratively, keeping your business logic clean.

## Deep Dive

### Basic Decorator

```python
def my_decorator(func):
    def wrapper(*args, **kwargs):
        print("Before function call")
        result = func(*args, **kwargs)
        print("After function call")
        return result
    return wrapper

@my_decorator
def say_hello(name):
    print(f"Hello, {name}")

# Equivalent to: say_hello = my_decorator(say_hello)
```

### Preserving Metadata with `functools.wraps`

```python
import functools

def my_decorator(func):
    @functools.wraps(func)  # Preserves __name__, __doc__, etc.
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper
```

### Decorators with Arguments

```python
def repeat(n):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            for _ in range(n):
                result = func(*args, **kwargs)
            return result
        return wrapper
    return decorator

@repeat(3)
def greet(name):
    print(f"Hello, {name}")
```

### Common Built-in Decorators

```python
class MyClass:
    @staticmethod
    def static_method():  # No self parameter
        pass
    
    @classmethod
    def class_method(cls):  # Receives class, not instance
        pass
    
    @property
    def computed(self):  # Accessed like an attribute
        return self._value
```

### Stacking Decorators

```python
@decorator_a
@decorator_b
def func():
    pass
# Equivalent to: func = decorator_a(decorator_b(func))
```

## Common Pitfalls
1. **Forgetting `functools.wraps`** -- Without it, the wrapper replaces the original function's `__name__` and `__doc__`, which breaks introspection, help(), and debugging tools.
2. **Not forwarding `*args` and `**kwargs`** -- If your wrapper has a fixed signature, it will break when the decorated function's signature changes. Always use `*args, **kwargs`.
3. **Confusing decorator vs decorator factory** -- `@repeat(3)` calls `repeat(3)` first, which returns the actual decorator. Writing `@repeat` without parentheses when arguments are expected causes a TypeError.

## Best Practices
1. **Always use `@functools.wraps(func)`** -- It is a one-line addition that preserves the decorated function's identity.
2. **Keep decorators focused on one concern** -- A decorator that logs, validates, and retries is doing too much. Write separate decorators and stack them.

## Summary
- Decorators wrap functions to add behavior without modifying source code.
- `@decorator` is shorthand for `func = decorator(func)`.
- Always use `@functools.wraps(func)` to preserve the original function's metadata.
- Parameterized decorators require an extra layer of nesting (a decorator factory).
- Stack decorators for composable, single-responsibility enhancements.

## Code Examples

**Timer decorator**

```python
import functools
import time

# Practical decorator: timing
def timer(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = func(*args, **kwargs)
        elapsed = time.perf_counter() - start
        print(f"{func.__name__} took {elapsed:.4f}s")
        return result
    return wrapper

@timer
def slow_function():
    time.sleep(1)
    return "Done"
```


## Resources

- [Decorator Glossary](https://docs.python.org/3.14/glossary.html#term-decorator) — Official Python 3.14 glossary entry for decorators with links to related documentation

---

> 📘 *This lesson is part of the [Python Fundamentals: Modern 3.14 Edition](https://stanza.dev/courses/python) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*