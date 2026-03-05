---
source_course: "python-performance"
source_lesson: "python-performance-ctypes-cffi"
---

# ctypes & cffi

## Introduction

Python's `ctypes` and the third-party `cffi` library let you call functions in compiled C libraries directly from Python, without writing any C extension code. This is the fastest path to leveraging existing native code. Python 3.14 updates ctypes with improved free-threading support and a new `dllist()` function.

## Key Concepts

- **ctypes**: A built-in Python module for loading shared libraries (.so/.dll/.dylib) and calling their functions with explicit type declarations for arguments and return values.
- **cffi**: A third-party library that parses C header declarations to automatically generate Python bindings, reducing boilerplate and type errors.
- **ABI mode vs API mode**: cffi's two modes -- ABI mode loads libraries at runtime (like ctypes), while API mode compiles a C wrapper for better performance and error checking.
- **Structures and Pointers**: Both ctypes and cffi support defining C structs, passing pointers, and managing memory layouts for complex data exchange.
- **dllist()**: New in Python 3.14, `ctypes.util.dllist()` lists all currently loaded shared libraries, useful for debugging and introspection.

## Real World Context

Many Python libraries use ctypes or cffi under the hood: `cryptography` uses cffi to bind to OpenSSL, `pyglet` uses ctypes to call OpenGL, and `pyaudio` uses ctypes to wrap PortAudio. When you need to call a system library or a vendor SDK that ships as a .so/.dll, ctypes and cffi are the tools to reach for.

## Deep Dive

### ctypes Basics: Loading a Library

The first step is loading the shared library:

```python
import ctypes
import ctypes.util

# Find the C math library
lib_path = ctypes.util.find_library('m')  # Returns 'libm.so.6' on Linux
lib = ctypes.CDLL(lib_path)

# Call sqrt - but we need to set types first!
lib.sqrt.argtypes = [ctypes.c_double]
lib.sqrt.restype = ctypes.c_double

result = lib.sqrt(144.0)
print(result)  # 12.0
```

Always declare `argtypes` and `restype`. Without them, ctypes assumes `c_int` for everything, causing silent data corruption.

### ctypes Structures

For complex data, define C-compatible structs:

```python
import ctypes

class Point(ctypes.Structure):
    _fields_ = [
        ('x', ctypes.c_double),
        ('y', ctypes.c_double),
    ]

class Rectangle(ctypes.Structure):
    _fields_ = [
        ('origin', Point),
        ('width', ctypes.c_double),
        ('height', ctypes.c_double),
    ]

# Create and use
rect = Rectangle(Point(0.0, 0.0), 100.0, 50.0)
print(f"Origin: ({rect.origin.x}, {rect.origin.y})")
print(f"Size: {rect.width} x {rect.height}")
```

### Python 3.14 ctypes Updates

Python 3.14 brings several improvements to ctypes:

```python
import ctypes.util

# New in 3.14: dllist() lists loaded shared libraries
loaded_libs = ctypes.util.dllist()
for lib in loaded_libs:
    print(lib)

# Free-threading support: ctypes is now safe to use
# in free-threaded Python (PEP 703) without the GIL
```

### cffi: ABI Mode (Runtime)

cffi reduces boilerplate by parsing C declarations:

```python
from cffi import FFI

ffi = FFI()

# Declare the C interface
ffi.cdef("""
    double sqrt(double x);
    double pow(double base, double exp);
""")

# Load the library (ABI mode)
lib = ffi.dlopen('m')  # libm

print(lib.sqrt(144.0))      # 12.0
print(lib.pow(2.0, 10.0))   # 1024.0
```

### cffi: API Mode (Compiled)

For better performance, cffi can compile a wrapper:

```python
from cffi import FFI

ffi = FFI()
ffi.cdef("double sqrt(double x);")

ffi.set_source("_math_ext",
    '#include <math.h>',
    libraries=['m']
)

ffi.compile()  # Generates _math_ext.so

# Then import like a regular module
from _math_ext import lib
print(lib.sqrt(144.0))
```

### When to Use Each

| Scenario | Use |
|----------|-----|
| Quick one-off library call | ctypes |
| Complex C API with many functions | cffi (API mode) |
| Need to distribute a package | cffi (API mode) |
| System/OS library calls | ctypes |
| OpenSSL, crypto libraries | cffi |
| Performance-critical bindings | cffi (API mode) or Cython |

## Common Pitfalls

1. **Forgetting to set argtypes/restype in ctypes** -- Without explicit type declarations, ctypes defaults to `c_int` for all arguments and return values, which silently truncates floats to 0 and corrupts pointers.
2. **Memory management across the boundary** -- When C code allocates memory, Python's garbage collector doesn't know about it. You must explicitly free C-allocated memory or risk memory leaks that grow unbounded.
3. **Platform-specific library names** -- Hard-coding `'libfoo.so'` breaks on macOS (.dylib) and Windows (.dll). Always use `ctypes.util.find_library('foo')` to locate libraries portably.

## Best Practices

1. **Always declare argument and return types** -- Set `argtypes` and `restype` for every ctypes function call to prevent silent data corruption.
2. **Prefer cffi API mode for distribution** -- API mode compiles a proper C extension that's faster and catches type errors at compile time rather than runtime.
3. **Use context managers for resource cleanup** -- Wrap C resource allocation/deallocation in Python context managers to ensure cleanup even when exceptions occur.

## Summary

- ctypes is built into Python and ideal for quick calls to existing C libraries.
- cffi provides cleaner syntax by parsing C header declarations and offers a compiled API mode.
- Python 3.14 adds `dllist()` to ctypes and improves free-threading support.
- Always declare argument types explicitly to prevent silent data corruption.
- Use `ctypes.util.find_library()` for portable library loading across operating systems.

## Code Examples

**Loading a shared library with ctypes, setting argument and return types explicitly to avoid silent data corruption**

```python
import ctypes
import ctypes.util

# Portably find and load the C math library
lib_path = ctypes.util.find_library('m')
if lib_path:
    libm = ctypes.CDLL(lib_path)

    # IMPORTANT: Always set argument and return types
    libm.sqrt.argtypes = [ctypes.c_double]
    libm.sqrt.restype = ctypes.c_double

    libm.pow.argtypes = [ctypes.c_double, ctypes.c_double]
    libm.pow.restype = ctypes.c_double

    print(f"sqrt(144) = {libm.sqrt(144.0)}")
    print(f"pow(2, 10) = {libm.pow(2.0, 10.0)}")

    # Without restype, this would return garbage!
    # libm.sqrt(144.0) would return 0 (truncated to int)
```

**Defining C-compatible structures with ctypes for passing complex data types between Python and C libraries**

```python
import ctypes

# Define a C-compatible struct
class Vector3(ctypes.Structure):
    _fields_ = [
        ('x', ctypes.c_float),
        ('y', ctypes.c_float),
        ('z', ctypes.c_float),
    ]

    def __repr__(self):
        return f"Vector3({self.x}, {self.y}, {self.z})"

# Create instances
v1 = Vector3(1.0, 2.0, 3.0)
v2 = Vector3(4.0, 5.0, 6.0)

# Arrays of structs
VectorArray = Vector3 * 3
points = VectorArray(
    Vector3(0, 0, 0),
    Vector3(1, 1, 1),
    Vector3(2, 2, 2),
)

for p in points:
    print(p)
```


## Resources

- [ctypes -- A foreign function library for Python](https://docs.python.org/3.14/library/ctypes.html) — Official ctypes documentation with full API reference
- [ctypes.util.dllist()](https://docs.python.org/3.14/library/ctypes.html#ctypes.util.dllist) — New in Python 3.14: list loaded shared libraries

---

> 📘 *This lesson is part of the [Python Performance: JIT & Internals](https://stanza.dev/courses/python-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*