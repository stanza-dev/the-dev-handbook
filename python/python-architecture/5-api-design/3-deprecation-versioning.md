---
source_course: "python-architecture"
source_lesson: "python-architecture-deprecation-versioning"
---

# Deprecation & Versioning

## Introduction

APIs evolve over time. Functions get renamed, parameters change, and entire modules get replaced. Handling these transitions gracefully is essential for maintaining trust with consumers of your code. Python provides built-in tools for deprecation warnings, and semantic versioning offers a framework for communicating the impact of changes.

## Key Concepts

- **`warnings.warn()`**: Python's built-in function to emit deprecation warnings without breaking existing code.
- **`DeprecationWarning`**: The standard warning category for features that will be removed in future versions.
- **`PendingDeprecationWarning`**: For features that will be deprecated in the future (two-stage deprecation).
- **Semantic Versioning (SemVer)**: A versioning scheme `MAJOR.MINOR.PATCH` that communicates the nature of changes.
- **Backwards compatibility**: The practice of ensuring existing code continues to work when the library is updated.

## Real World Context

NumPy, Django, and the Python standard library all use `DeprecationWarning` extensively. For example, when NumPy renamed `np.bool` to `np.bool_`, they first issued `DeprecationWarning` for two major versions before removing `np.bool` entirely. This gave millions of users time to migrate. Django follows a strict deprecation policy: features are deprecated in version N, emit warnings in N+1, and are removed in N+2.

## Deep Dive

### Basic deprecation warning

```python
import warnings

def old_function(x: int) -> int:
    """Deprecated: use new_function instead."""
    warnings.warn(
        "old_function() is deprecated, use new_function() instead.",
        DeprecationWarning,
        stacklevel=2,  # Points to the caller, not this function
    )
    return new_function(x)

def new_function(x: int) -> int:
    return x * 2
```

### The `stacklevel` parameter

`stacklevel=2` is critical — it makes the warning point to the *caller's* code, not to the deprecated function itself. Without it, the warning message shows the line inside `old_function`, which is useless to the consumer.

```python
# With stacklevel=2:
# UserWarning: old_function() is deprecated
#   result = old_function(5)    <-- points here (caller)

# With stacklevel=1 (default):
# UserWarning: old_function() is deprecated
#   warnings.warn(...)          <-- points here (useless)
```

### Deprecation decorator

```python
import warnings
import functools
from typing import Callable, Any

def deprecated(message: str) -> Callable:
    """Decorator to mark a function as deprecated."""
    def decorator(func: Callable) -> Callable:
        @functools.wraps(func)
        def wrapper(*args: Any, **kwargs: Any) -> Any:
            warnings.warn(
                f"{func.__name__}() is deprecated. {message}",
                DeprecationWarning,
                stacklevel=2,
            )
            return func(*args, **kwargs)
        return wrapper
    return decorator

@deprecated("Use process_v2() instead.")
def process(data: str) -> str:
    return data.upper()
```

### Deprecating function parameters

```python
def fetch_data(
    url: str,
    timeout: int = 30,
    verify: bool | None = None,  # New parameter
    verify_ssl: bool | None = None,  # Deprecated parameter
) -> bytes:
    if verify_ssl is not None:
        warnings.warn(
            "verify_ssl is deprecated, use verify instead.",
            DeprecationWarning,
            stacklevel=2,
        )
        if verify is None:
            verify = verify_ssl
    verify = verify if verify is not None else True
    # ... fetch logic
    return b'data'
```

### Semantic versioning

```
MAJOR.MINOR.PATCH
  |     |     |
  |     |     +-- Bug fixes (backwards compatible)
  |     +-------- New features (backwards compatible)
  +-------------- Breaking changes
```

```python
# __version__ in your package
__version__ = "2.3.1"

# Version tuple for programmatic access
VERSION = (2, 3, 1)

# Deprecation timeline example:
# v2.3: old_func() emits DeprecationWarning
# v2.4: old_func() emits DeprecationWarning (reminder in changelog)
# v3.0: old_func() is removed (MAJOR bump = breaking change)
```

### Controlling warning visibility

```python
import warnings

# Show all deprecation warnings (useful in tests)
warnings.simplefilter('always', DeprecationWarning)

# Turn warnings into errors (useful in CI)
warnings.simplefilter('error', DeprecationWarning)

# Ignore specific warnings
warnings.filterwarnings(
    'ignore',
    message='old_function.*deprecated',
    category=DeprecationWarning,
)
```

## Common Pitfalls

1. **Forgetting `stacklevel=2`**: The warning points to the inside of the deprecated function instead of the caller's code, making it hard to find where to fix.
2. **Removing without deprecation period**: Deleting a function in a minor release breaks consumers. Always deprecate first, remove in the next major version.
3. **Silent deprecation**: Only documenting deprecation in release notes but not emitting runtime warnings. Users who do not read changelogs will be surprised.

## Best Practices

- Use `DeprecationWarning` for features to be removed in the next major version.
- Use `PendingDeprecationWarning` for two-stage deprecation (will deprecate in a future minor version, remove in the next major).
- Always include a migration path in the warning message ("use X instead").
- In your test suite, set `warnings.simplefilter('error', DeprecationWarning)` to catch accidental use of deprecated APIs.
- Document the deprecation timeline in your changelog.

## Summary

Python's `warnings` module provides `DeprecationWarning` to signal that a function, parameter, or module will be removed in a future version. Combined with semantic versioning, this gives consumers a clear migration path. Always use `stacklevel=2`, always provide an alternative in the warning message, and always follow through by removing deprecated code in the next major release.

## Code Examples

**Reusable deprecation decorator with warning**

```python
import warnings
import functools
from typing import Callable, Any

def deprecated(message: str) -> Callable:
    """Decorator to mark functions as deprecated."""
    def decorator(func: Callable) -> Callable:
        @functools.wraps(func)
        def wrapper(*args: Any, **kwargs: Any) -> Any:
            warnings.warn(
                f"{func.__name__}() is deprecated. {message}",
                DeprecationWarning,
                stacklevel=2,
            )
            return func(*args, **kwargs)
        return wrapper
    return decorator

@deprecated("Use calculate_total_v2() instead.")
def calculate_total(items: list[float]) -> float:
    return sum(items)

# Calling it emits:
# DeprecationWarning: calculate_total() is deprecated.
# Use calculate_total_v2() instead.
result = calculate_total([1.0, 2.0, 3.0])
```

**Turning deprecation warnings into test errors**

```python
import warnings

# In your test configuration (conftest.py or setUp)
warnings.simplefilter('error', DeprecationWarning)

# Now any deprecated call raises an exception:
try:
    result = calculate_total([1.0, 2.0])  # Raises!
except DeprecationWarning as e:
    print(f"Caught: {e}")

# In production, warnings are shown but don't crash:
warnings.simplefilter('default', DeprecationWarning)
result = calculate_total([1.0, 2.0])  # Works, but prints warning
```


## Resources

- [warnings — Warning control](https://docs.python.org/3/library/warnings.html) — Official Python docs on the warnings module
- [PEP 387 – Backwards Compatibility Policy](https://peps.python.org/pep-0387/) — Python's official backwards compatibility and deprecation policy
- [Semantic Versioning 2.0.0](https://semver.org/) — The SemVer specification for version numbering

---

> 📘 *This lesson is part of the [Python Architecture: Patterns & Type System](https://stanza.dev/courses/python-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*