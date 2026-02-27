---
source_course: "python-metaprogramming"
source_lesson: "python-metaprogramming-decorator-basics"
---

# Decorator Fundamentals

## Introduction

Decorators are one of Python's most elegant features, letting you modify or extend the behavior of functions and classes without changing their source code. They are used everywhere in modern Python -- from Flask routes to pytest fixtures to dataclass fields. Understanding how decorators work under the hood is essential for writing clean, reusable code.

## Key Concepts

- **Decorator** -- A callable (usually a function) that takes another function as input and returns a new function with modified or extended behavior.
- **`@` syntax** -- Syntactic sugar that applies a decorator to the function defined immediately below it.
- **Wrapper function** -- The inner function returned by a decorator that replaces the original function at the call site.
- **`*args, **kwargs`** -- The universal signature pattern that lets a wrapper accept and forward any combination of positional and keyword arguments.

## Real World Context

Decorators are a daily tool for working Python developers. Web frameworks like Flask and FastAPI use `@app.route("/path")` to map functions to HTTP endpoints. Testing libraries use `@pytest.fixture` and `@unittest.mock.patch` to set up test state. Logging, caching, authentication checks, and retry logic are all commonly implemented as decorators. Once you understand the pattern, you can write your own decorators to eliminate repetitive boilerplate across your codebase.

## Deep Dive

At its core, a decorator is just a function that takes a function and returns a function. The `@` symbol is syntactic sugar that makes this pattern readable. The following two blocks of code are exactly equivalent:

```python
@decorator
def func():
    pass

# Is equivalent to:
def func():
    pass
func = decorator(func)
```

The `@decorator` line tells Python: after defining `func`, immediately pass it to `decorator()` and rebind the name `func` to whatever `decorator()` returns.

The most common decorator pattern defines an inner wrapper function that runs code before and/or after calling the original function:

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

When `greet("Alice")` is called, it actually calls `wrapper("Alice")`, which prints "Before", then delegates to the original `greet`, then prints "After". The decorator has added behavior without modifying the original function.

Notice that the wrapper uses `*args` and `**kwargs`. This is critical because the wrapper must accept whatever arguments the original function expects. Without this, the decorator would only work for functions with a specific signature:

```python
def universal_decorator(func):
    def wrapper(*args, **kwargs):  # Accept anything
        return func(*args, **kwargs)  # Pass everything through
    return wrapper
```

By forwarding `*args` and `**kwargs`, the wrapper works with any function regardless of its parameter list.

Python also supports stacking multiple decorators on a single function. Decorators are applied bottom-up, meaning the one closest to the function runs first:

```python
@decorator_a
@decorator_b
def func():
    pass

# Is equivalent to:
func = decorator_a(decorator_b(func))
# decorator_b runs first, then decorator_a wraps the result
```

This ordering matters because each decorator wraps the result of the one below it. The outermost decorator (`decorator_a`) controls the final behavior at call time.

## Common Pitfalls

- **Forgetting to return the result.** If your wrapper calls `func(*args, **kwargs)` but does not `return` the result, the decorated function will always return `None`, silently discarding the original return value.
- **Not using `*args, **kwargs`.** If you hardcode the wrapper's parameter list (e.g., `def wrapper(name):`), the decorator breaks as soon as someone uses it on a function with a different signature.
- **Confusing decorator application order.** With stacked decorators, the bottom decorator is applied first. Writing `@a @b @c` means `f = a(b(c(f)))`, not `f = c(b(a(f)))`.

## Best Practices

- Always use `*args, **kwargs` in your wrapper so the decorator is reusable across functions with different signatures.
- Always `return` the result of calling `func(*args, **kwargs)` inside the wrapper to preserve the original function's return value.
- Use `functools.wraps` on the wrapper (covered in the next lesson) to preserve the original function's metadata like `__name__` and `__doc__`.

## Summary

- A decorator is a function that takes a function and returns a modified version of it.
- The `@decorator` syntax is equivalent to `func = decorator(func)`.
- Wrapper functions should use `*args, **kwargs` to forward all arguments transparently.
- Multiple decorators are applied bottom-up: the closest decorator to the function runs first.
- Always return the result from the wrapped call to avoid silently dropping return values.

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


## Resources

- [Decorators — Python Glossary](https://docs.python.org/3.14/glossary.html#term-decorator) — Official Python glossary entry for decorators
- [PEP 318 — Decorators for Functions and Methods](https://peps.python.org/pep-0318/) — The PEP that introduced decorator syntax

---

> 📘 *This lesson is part of the [Python Metaprogramming & Introspection](https://stanza.dev/courses/python-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*