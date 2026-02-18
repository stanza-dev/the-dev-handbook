---
source_course: "python-performance"
source_lesson: "python-performance-ctypes-cffi"
---

# Calling C from Python

## ctypes (Standard Library)

```python
import ctypes

# Load shared library
libc = ctypes.CDLL('libc.so.6')  # Linux
# libc = ctypes.CDLL('libc.dylib')  # macOS

# Call C function
libc.printf(b"Hello from C!\n")

# With typed arguments
libc.strlen.argtypes = [ctypes.c_char_p]
libc.strlen.restype = ctypes.c_size_t
result = libc.strlen(b"hello")
print(result)  # 5
```

## Defining Structures

```python
import ctypes

class Point(ctypes.Structure):
    _fields_ = [
        ('x', ctypes.c_double),
        ('y', ctypes.c_double)
    ]

p = Point(10.0, 20.0)
print(p.x, p.y)
```

## cffi (Third-party, more Pythonic)

```python
from cffi import FFI

ffi = FFI()

# Declare C functions
ffi.cdef("""
    int printf(const char *format, ...);
    size_t strlen(const char *s);
""")

# Load library
C = ffi.dlopen(None)

# Call functions
result = C.strlen(b"hello")
print(result)  # 5
```

## When to Use Each

- **ctypes**: Quick one-off calls, no build step
- **cffi**: Larger projects, better C compatibility

## Code Examples

**Calling qsort from C**

```python
import ctypes
import platform

# Cross-platform library loading
if platform.system() == 'Windows':
    lib = ctypes.CDLL('msvcrt')
else:
    lib = ctypes.CDLL(None)  # Load C library

# Call qsort (C standard library)
def compare(a, b):
    return a[0] - b[0]

COMPARE_FUNC = ctypes.CFUNCTYPE(ctypes.c_int, 
                                 ctypes.POINTER(ctypes.c_int),
                                 ctypes.POINTER(ctypes.c_int))

arr = (ctypes.c_int * 5)(5, 2, 8, 1, 9)
lib.qsort(arr, len(arr), ctypes.sizeof(ctypes.c_int), 
          COMPARE_FUNC(compare))
print(list(arr))  # [1, 2, 5, 8, 9]
```


---

> 📘 *This lesson is part of the [Python Performance: JIT & Internals](https://stanza.dev/courses/python-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*