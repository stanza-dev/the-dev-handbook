---
source_course: "python-architecture"
source_lesson: "python-architecture-api-design-all"
---

# Public APIs & `__all__`

## Introduction

Every Python module exposes a public interface — the set of names that other code should use. Without explicit control, consumers may depend on internal implementation details, making refactoring dangerous. The `__all__` variable is Python's built-in mechanism for declaring a module's public API.

## Key Concepts

- **`__all__`**: A list of strings defining which names are exported by `from module import *`.
- **Underscore convention**: Names starting with `_` are considered private by convention, but are still importable.
- **Star import filtering**: Only names in `__all__` are included in wildcard imports.
- **Direct imports are unaffected**: `from module import _private_func` always works, regardless of `__all__`.

## Real World Context

Libraries like `requests`, `flask`, and `pandas` all use `__all__` to curate their public surface. When you type `from requests import *`, you get exactly the names the library authors intended — `get`, `post`, `Session`, etc. — not internal helpers like `_internal_utils`. This is critical for libraries with hundreds of internal functions that should never be called directly.

## Deep Dive

### Defining `__all__`

```python
# mylib/utils.py
__all__ = ['PublicClass', 'public_func']

class PublicClass:
    """Part of the public API."""
    pass

def public_func() -> str:
    """Part of the public API."""
    return _helper()

class _PrivateClass:
    """Not exported via star import."""
    pass

def _helper() -> str:
    """Internal implementation detail."""
    return "internal"
```

### How star imports work

```python
# Without __all__: imports everything without leading underscore
from mylib.utils import *
# PublicClass, public_func are available
# _PrivateClass, _helper are NOT (underscore convention)

# With __all__: imports ONLY what's listed
from mylib.utils import *
# PublicClass, public_func are available
# Even non-underscore names not in __all__ would be excluded
```

### Combining `__all__` with ABCs

Abstract Base Classes (`abc.ABC`) enforce interface contracts at the class level. Combined with `__all__`, you can expose clean interfaces while hiding implementation:

```python
from abc import ABC, abstractmethod

__all__ = ['Stream']

class Stream(ABC):
    @abstractmethod
    def read(self) -> bytes:
        """Read data from the stream."""
        ...

    @abstractmethod
    def write(self, data: bytes) -> int:
        """Write data to the stream."""
        ...
```

## Common Pitfalls

1. **Forgetting to update `__all__`**: When you add a new public function, you must also add it to `__all__`. Linters like `pylint` can warn about this.
2. **Relying solely on underscores**: The underscore convention is just a convention — it does not prevent imports. Use `__all__` for enforceable API boundaries.
3. **Putting too much in `__all__`**: Only export what consumers genuinely need. A smaller public API is easier to maintain and version.

## Best Practices

- Always define `__all__` in library modules, especially `__init__.py` files.
- Keep `__all__` at the top of the file, right after imports, so it serves as a quick reference.
- Use `_` prefix for internal helpers as a secondary signal alongside `__all__`.
- Run `pyflakes` or `ruff` to detect names in `__all__` that do not exist in the module.

## Summary

`__all__` is the standard mechanism for declaring a module's public API. It controls `from module import *` behavior, acts as documentation for consumers, and helps tools enforce API boundaries. Combined with the underscore convention and ABCs, it provides a robust approach to interface design in Python.

## Code Examples

**Using __all__ to curate a package's public API**

```python
# analytics/__init__.py
__all__ = ['track_event', 'get_metrics']

from ._tracker import track_event
from ._metrics import get_metrics
from ._internal import _flush_buffer  # NOT in __all__

# Consumer code
from analytics import *
track_event('signup')   # Works
get_metrics()           # Works
# _flush_buffer          # NameError if only star-imported
```

**Abstract Base Class with __all__ for interface isolation**

```python
from abc import ABC, abstractmethod

__all__ = ['Stream']

class Stream(ABC):
    @abstractmethod
    def read(self) -> bytes:
        ...

    @abstractmethod
    def write(self, data: bytes) -> int:
        ...

class _BufferedStream(Stream):
    """Internal implementation, not exported."""
    def read(self) -> bytes:
        return b'data'

    def write(self, data: bytes) -> int:
        return len(data)
```


## Resources

- [Python import system - __all__](https://docs.python.org/3/reference/simple_stmts.html#the-import-statement) — Official Python docs on import mechanics and __all__
- [abc — Abstract Base Classes](https://docs.python.org/3/library/abc.html) — Official Python docs on ABC and abstractmethod

---

> 📘 *This lesson is part of the [Python Architecture: Patterns & Type System](https://stanza.dev/courses/python-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*