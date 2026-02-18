---
source_course: "python"
source_lesson: "python-context-managers"
---

# Context Managers

Context managers ensure proper acquisition and release of resources using the `with` statement.

## Using Context Managers

```python
# File handling
with open('file.txt', 'r') as f:
    content = f.read()
# File is automatically closed, even if exception occurs

# Multiple context managers
with open('in.txt') as src, open('out.txt', 'w') as dst:
    dst.write(src.read())
```

## Creating Context Managers (Class-based)

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

## Creating Context Managers (Decorator-based)

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

## contextlib Utilities

```python
from contextlib import suppress, nullcontext

# Suppress specific exceptions
with suppress(FileNotFoundError):
    os.remove('temp.txt')  # No error if file doesn't exist

# Conditional context manager
with (lock if threaded else nullcontext()):
    process()
```

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


---

> 📘 *This lesson is part of the [Python Fundamentals: Modern 3.15 Edition](https://stanza.dev/courses/python) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*