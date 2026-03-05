---
source_course: "python-performance"
source_lesson: "python-performance-object-model"
---

# Python Object Model

## Introduction

In Python, everything is an object: integers, strings, functions, classes, and even `None`. Each object carries overhead for reference counting and type information, and understanding this overhead is essential for writing memory-efficient Python. This lesson explores the internal structure of Python objects, built-in memory optimizations like the small integer cache, and how to measure object sizes.

## Key Concepts

- **PyObject**: The C struct underlying every Python object, containing at minimum a reference count (`ob_refcnt`) and a type pointer (`ob_type`).
- **Reference Count**: An integer stored in every object that tracks how many references point to it. When it reaches zero, the object is immediately deallocated.
- **Small Integer Cache**: CPython pre-allocates and reuses integer objects in the range -5 to 256, so these values are singletons.
- **String Interning**: CPython may reuse the same string object for identical short strings, especially those that look like identifiers.
- **`sys.getsizeof()`**: Returns the size in bytes of a single object (shallow size), not including the sizes of objects it references.

## Real World Context

When you create a list of one million dictionaries to process API responses, each dictionary and its keys consume memory far beyond the raw data they hold. Understanding PyObject overhead helps you choose between dictionaries, named tuples, dataclasses with `__slots__`, or arrays depending on your memory constraints.

## Deep Dive

### The PyObject Structure

Every Python object in CPython is backed by a C struct. At minimum, it contains:

```c
// Simplified PyObject (CPython internals)
typedef struct {
    Py_ssize_t ob_refcnt;    // 8 bytes on 64-bit
    PyTypeObject *ob_type;   // 8 bytes on 64-bit
    // ... object-specific data follows
} PyObject;
```

This means the absolute minimum overhead for any Python object on a 64-bit system is **16 bytes** — just for the refcount and type pointer, before any actual data.

### Object Sizes in Practice

```python
import sys

# Minimum object sizes on 64-bit CPython 3.14
print(f"int(0):    {sys.getsizeof(0)} bytes")       # 28 bytes
print(f"int(1):    {sys.getsizeof(1)} bytes")       # 28 bytes
print(f"float:     {sys.getsizeof(1.0)} bytes")     # 24 bytes
print(f"bool:      {sys.getsizeof(True)} bytes")    # 28 bytes
print(f"None:      {sys.getsizeof(None)} bytes")    # 16 bytes
print(f"empty str: {sys.getsizeof('')} bytes")      # 49 bytes
print(f"empty list:{sys.getsizeof([])} bytes")      # 56 bytes
print(f"empty dict:{sys.getsizeof({})} bytes")      # 64 bytes
print(f"empty set: {sys.getsizeof(set())} bytes")   # 216 bytes
```

Note that `sys.getsizeof()` returns the **shallow** size. A list's size does not include the objects it contains.

### The Small Integer Cache

CPython pre-creates integer objects for -5 through 256. Every time your code uses one of these values, Python returns the same object:

```python
# Cached range: -5 to 256
a = 256
b = 256
print(a is b)  # True — same object from the cache

a = 257
b = 257
print(a is b)  # False — different objects created

# This also works with negative numbers
a = -5
b = -5
print(a is b)  # True

a = -6
b = -6
print(a is b)  # False
```

This optimization saves enormous amounts of memory because small integers appear everywhere in Python programs (loop counters, indices, boolean-like flags).

### String Interning

CPython automatically interns strings that look like identifiers (letters, digits, underscores only). You can also force interning:

```python
import sys

# Automatically interned (looks like an identifier)
a = "hello"
b = "hello"
print(a is b)  # True

# Not automatically interned (contains space)
a = "hello world"
b = "hello world"
print(a is b)  # May be False

# Force interning for any string
a = sys.intern("hello world")
b = sys.intern("hello world")
print(a is b)  # True — guaranteed same object
```

Interning is useful when you have thousands of identical string keys (e.g., JSON field names) that would otherwise waste memory as duplicate objects.

### Deep Size Calculation

To measure the total memory of an object including everything it references, you need a recursive function:

```python
import sys

def deep_getsizeof(obj, seen=None):
    """Recursively calculate total memory of an object."""
    if seen is None:
        seen = set()
    obj_id = id(obj)
    if obj_id in seen:
        return 0
    seen.add(obj_id)

    size = sys.getsizeof(obj)

    if isinstance(obj, dict):
        size += sum(
            deep_getsizeof(k, seen) + deep_getsizeof(v, seen)
            for k, v in obj.items()
        )
    elif isinstance(obj, (list, tuple, set, frozenset)):
        size += sum(deep_getsizeof(item, seen) for item in obj)

    return size

data = {"users": [{"name": "Alice", "age": 30}]}
print(f"Shallow: {sys.getsizeof(data)} bytes")
print(f"Deep:    {deep_getsizeof(data)} bytes")
```

## Common Pitfalls

- **Using `is` to compare values instead of `==`**: The small integer cache makes `a is b` work for small numbers, but this is an implementation detail. Always use `==` for value comparison. `is` should only be used for `None` checks.
- **Trusting `sys.getsizeof()` for total memory**: `getsizeof()` only reports the shallow size of a single object. A dictionary containing a million strings will report only ~64 bytes for the dict itself.
- **Assuming all strings are interned**: Only identifier-like strings are automatically interned. If you need guaranteed interning for non-identifier strings, use `sys.intern()` explicitly.

## Best Practices

- **Use `sys.getsizeof()` with a deep helper for accurate measurements**: When investigating memory usage, always account for nested objects with a recursive size calculation.
- **Leverage `__slots__` for classes with many instances**: Eliminating the per-instance `__dict__` saves 100+ bytes per object, which matters when you have millions of instances.
- **Use `sys.intern()` for repeated string keys**: When processing data with thousands of identical string keys (e.g., column headers, JSON fields), interning prevents duplicate string objects.

## Summary

- Every Python object carries at least 16 bytes of overhead on 64-bit systems (8 bytes for reference count + 8 bytes for type pointer)
- CPython caches integer objects from -5 to 256 as singletons, saving memory for commonly used values
- `sys.getsizeof()` reports only shallow size; use a recursive helper for total memory including nested objects
- String interning automatically reuses identifier-like strings; use `sys.intern()` to force interning for other strings
- Understanding object overhead informs choices between dicts, named tuples, `__slots__` classes, and arrays for memory-critical applications

## Code Examples

**Measuring shallow versus deep memory usage of nested data structures and examining the per-type overhead of common Python objects**

```python
import sys

def deep_getsizeof(obj, seen=None):
    """Recursively calculate total memory including nested objects."""
    if seen is None:
        seen = set()
    obj_id = id(obj)
    if obj_id in seen:
        return 0
    seen.add(obj_id)
    size = sys.getsizeof(obj)
    if isinstance(obj, dict):
        size += sum(
            deep_getsizeof(k, seen) + deep_getsizeof(v, seen)
            for k, v in obj.items()
        )
    elif isinstance(obj, (list, tuple, set, frozenset)):
        size += sum(deep_getsizeof(item, seen) for item in obj)
    return size

# Compare shallow vs deep sizes
users = [
    {"name": "Alice", "role": "admin", "score": 95},
    {"name": "Bob", "role": "user", "score": 82},
]
print(f"Shallow list size: {sys.getsizeof(users)} bytes")
print(f"Deep total size:   {deep_getsizeof(users)} bytes")
print(f"\nPer-type overhead:")
print(f"  int(0):   {sys.getsizeof(0)} bytes")
print(f"  float:    {sys.getsizeof(0.0)} bytes")
print(f"  empty '': {sys.getsizeof('')} bytes")
print(f"  None:     {sys.getsizeof(None)} bytes")
```


## Resources

- [sys.getsizeof — System-specific parameters](https://docs.python.org/3.14/library/sys.html#sys.getsizeof) — Official documentation for sys.getsizeof() and its limitations
- [sys.intern — String interning](https://docs.python.org/3.14/library/sys.html#sys.intern) — Official documentation for sys.intern() used to intern strings for memory optimization

---

> 📘 *This lesson is part of the [Python Performance: JIT & Internals](https://stanza.dev/courses/python-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*