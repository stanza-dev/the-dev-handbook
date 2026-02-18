---
source_course: "python-metaprogramming"
source_lesson: "python-metaprogramming-decorator-basics"
---

# Understanding Decorators

## What is a Decorator?

A decorator is a function that modifies another function. The `@` syntax is just sugar:

```python
@decorator
def func():
    pass

# Is equivalent to:
def func():
    pass
func = decorator(func)
```

## Basic Decorator Pattern

```python
def my_decorator(func):
    def wrapper(*args, **kwargs):
        print("Before")
        result = func(*args, **kwargs)
        print("After")
        return result
    return wrapper

@my_decorator
def greet(name):
    print(f"Hello, {name}")

greet("Alice")
# Before
# Hello, Alice
# After
```

## Why *args and **kwargs?

The wrapper must accept any arguments the original function might receive:

```python
def universal_decorator(func):
    def wrapper(*args, **kwargs):  # Accept anything
        return func(*args, **kwargs)  # Pass everything through
    return wrapper
```

## Multiple Decorators

```python
@decorator_a
@decorator_b
def func():
    pass

# Is equivalent to:
func = decorator_a(decorator_b(func))
# decorator_b runs first, then decorator_a wraps the result
```

## Code Examples

**Timer decorator**

```python
def timer(func):
    import time
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = func(*args, **kwargs)
        elapsed = time.perf_counter() - start
        print(f"{func.__name__} took {elapsed:.4f}s")
        return result
    return wrapper

@timer
def slow_function():
    import time
    time.sleep(1)
    return "Done"

slow_function()  # Prints: slow_function took 1.00XXs
```


---

> 📘 *This lesson is part of the [Python Metaprogramming & Introspection](https://stanza.dev/courses/python-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*