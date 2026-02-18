---
source_course: "python"
source_lesson: "python-decorators"
---

# Understanding Decorators

Decorators modify or enhance functions without changing their code.

## Basic Decorator

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

## Preserving Metadata with `functools.wraps`

```python
import functools

def my_decorator(func):
    @functools.wraps(func)  # Preserves __name__, __doc__, etc.
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper
```

## Decorators with Arguments

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

## Common Built-in Decorators

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

## Stacking Decorators

```python
@decorator_a
@decorator_b
def func():
    pass
# Equivalent to: func = decorator_a(decorator_b(func))
```

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


---

> 📘 *This lesson is part of the [Python Fundamentals: Modern 3.15 Edition](https://stanza.dev/courses/python) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*