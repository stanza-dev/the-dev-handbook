---
source_course: "python-performance"
source_lesson: "python-performance-object-model"
---

# How Python Stores Objects

## Everything is an Object

```python
# Even simple values are objects
x = 42
print(type(x))   # <class 'int'>
print(id(x))     # Memory address
print(x.__class__)  # int
```

## Object Structure (CPython)

Every Python object has:

```c
// Simplified PyObject structure
struct PyObject {
    Py_ssize_t ob_refcnt;  // Reference count
    PyTypeObject *ob_type;  // Type pointer
    // ... object data
};
```

Minimum overhead: 16 bytes (64-bit) + object data

## Small Integer Cache

```python
# Python caches small integers (-5 to 256)
a = 256
b = 256
print(a is b)  # True (same object)

a = 257
b = 257
print(a is b)  # False (different objects)
```

## String Interning

```python
# Short strings may be interned
a = "hello"
b = "hello"
print(a is b)  # True (interned)

a = "hello world!"
b = "hello world!"
print(a is b)  # May be False

# Force interning
import sys
a = sys.intern("any string")
b = sys.intern("any string")
print(a is b)  # True
```

## Checking Memory Size

```python
import sys

print(sys.getsizeof(1))        # 28 bytes
print(sys.getsizeof([]))       # 56 bytes
print(sys.getsizeof({}))       # 64 bytes
print(sys.getsizeof(""))       # 49 bytes
```

## Code Examples

**Deep memory size calculation**

```python
import sys

def deep_getsizeof(obj, seen=None):
    """Calculate total memory including nested objects."""
    if seen is None:
        seen = set()
    
    obj_id = id(obj)
    if obj_id in seen:
        return 0
    seen.add(obj_id)
    
    size = sys.getsizeof(obj)
    
    if isinstance(obj, dict):
        size += sum(deep_getsizeof(k, seen) + deep_getsizeof(v, seen)
                    for k, v in obj.items())
    elif isinstance(obj, (list, tuple, set, frozenset)):
        size += sum(deep_getsizeof(i, seen) for i in obj)
    
    return size

data = {'users': [{'name': 'Alice'}, {'name': 'Bob'}]}
print(f"Shallow: {sys.getsizeof(data)} bytes")
print(f"Deep: {deep_getsizeof(data)} bytes")
```


---

> 📘 *This lesson is part of the [Python Performance: JIT & Internals](https://stanza.dev/courses/python-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*