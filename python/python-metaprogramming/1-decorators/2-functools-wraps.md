---
source_course: "python-metaprogramming"
source_lesson: "python-metaprogramming-functools-wraps"
---

# The functools.wraps Problem

## The Problem

```python
def my_decorator(func):
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper

@my_decorator
def greet(name):
    """Greet someone."""
    return f"Hello, {name}"

print(greet.__name__)  # "wrapper" - Not "greet"!
print(greet.__doc__)   # None - Docstring lost!
```

## The Solution: functools.wraps

```python
import functools

def my_decorator(func):
    @functools.wraps(func)  # Copy metadata from func to wrapper
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper

@my_decorator
def greet(name):
    """Greet someone."""
    return f"Hello, {name}"

print(greet.__name__)  # "greet" - Correct!
print(greet.__doc__)   # "Greet someone." - Preserved!
```

## What wraps Copies

- `__name__`: Function name
- `__doc__`: Docstring
- `__module__`: Module name
- `__qualname__`: Qualified name
- `__annotations__`: Type annotations
- `__dict__`: Function attributes
- `__wrapped__`: Reference to original function

## Code Examples

**Complete decorator with wraps**

```python
import functools

def debug(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__}({args}, {kwargs})")
        result = func(*args, **kwargs)
        print(f"{func.__name__} returned {result}")
        return result
    return wrapper

@debug
def add(a: int, b: int) -> int:
    """Add two numbers."""
    return a + b

# Metadata preserved
print(add.__name__)        # add
print(add.__doc__)         # Add two numbers.
print(add.__annotations__) # {'a': int, 'b': int, 'return': int}
print(add.__wrapped__)     # <function add at 0x...>
```


---

> 📘 *This lesson is part of the [Python Metaprogramming & Introspection](https://stanza.dev/courses/python-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*