---
source_course: "python"
source_lesson: "python-context-managers"
---

# Context Managers

## Introduction
Context managers automate resource acquisition and release -- opening and closing files, acquiring and releasing locks, connecting and disconnecting from databases. The `with` statement guarantees cleanup even when exceptions occur.

## Key Concepts
- **`with` statement**: Enters a context manager on entry and calls its cleanup on exit, regardless of exceptions.
- **`__enter__` / `__exit__`**: The two methods a class must implement to work as a context manager.
- **`@contextmanager`**: A decorator from `contextlib` that lets you write context managers as generator functions.
- **`suppress()` / `nullcontext()`**: Utility context managers for common patterns.

## Real World Context
Almost every resource in professional Python code should be managed with a context manager: file handles, database connections, network sockets, locks, and temporary directories. Without them, resource leaks accumulate under error conditions, leading to file descriptor exhaustion, connection pool starvation, and hard-to-reproduce production failures.

## Deep Dive

### Using Context Managers

```python
# File handling
with open('file.txt', 'r') as f:
    content = f.read()
# File is automatically closed, even if exception occurs

# Multiple context managers
with open('in.txt') as src, open('out.txt', 'w') as dst:
    dst.write(src.read())
```

### Creating Context Managers (Class-based)

```python
class DatabaseConnection:
    def __init__(self, host):
        self.host = host
        self.connection = None
    
    def __enter__(self):
        self.connection = connect(self.host)
        return self.connection
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        self.connection.close()
        return False  # Don't suppress exceptions

with DatabaseConnection('localhost') as conn:
    conn.query('SELECT * FROM users')
```

### Creating Context Managers (Decorator-based)

```python
from contextlib import contextmanager

@contextmanager
def timer(label):
    start = time.perf_counter()
    try:
        yield  # Code in 'with' block runs here
    finally:
        elapsed = time.perf_counter() - start
        print(f"{label}: {elapsed:.3f}s")

with timer("Processing"):
    heavy_computation()
```

### contextlib Utilities

```python
from contextlib import suppress, nullcontext

# Suppress specific exceptions
with suppress(FileNotFoundError):
    os.remove('temp.txt')  # No error if file doesn't exist

# Conditional context manager
with (lock if threaded else nullcontext()):
    process()
```

## Common Pitfalls
1. **Returning `True` from `__exit__` accidentally** -- Returning `True` suppresses exceptions. Unless you intentionally want to swallow errors, always return `False` or `None`.
2. **Forgetting the `try/finally` in a `@contextmanager` function** -- Without it, cleanup code after `yield` will not run if the body of the `with` block raises an exception.
3. **Using context managers for objects that need to outlive the `with` block** -- If you need a database connection across multiple functions, pass it explicitly rather than reopening it in each `with` block.

## Best Practices
1. **Use `with` for every resource that needs cleanup** -- Files, sockets, locks, and temporary directories should all be context-managed.
2. **Prefer `@contextmanager` for simple cases** -- It is less boilerplate than writing a full class with `__enter__` and `__exit__`.

## Summary
- The `with` statement guarantees resource cleanup even when exceptions occur.
- Class-based context managers implement `__enter__` and `__exit__`.
- `@contextmanager` from `contextlib` turns generator functions into context managers with less boilerplate.
- `suppress()` silences specific exceptions; `nullcontext()` is a no-op placeholder.
- Always wrap cleanup in `try/finally` when using `@contextmanager`.

## Code Examples

**Practical context manager**

```python
from contextlib import contextmanager

@contextmanager
def temporary_chdir(path):
    """Temporarily change directory."""
    old_dir = os.getcwd()
    try:
        os.chdir(path)
        yield
    finally:
        os.chdir(old_dir)

with temporary_chdir('/tmp'):
    print(os.getcwd())  # /tmp
print(os.getcwd())  # Back to original
```


## Resources

- [contextlib Module](https://docs.python.org/3.14/library/contextlib.html) — Official Python 3.14 reference for context manager utilities and the contextmanager decorator

---

> 📘 *This lesson is part of the [Python Fundamentals: Modern 3.14 Edition](https://stanza.dev/courses/python) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*