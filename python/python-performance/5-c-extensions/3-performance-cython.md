---
source_course: "python-performance"
source_lesson: "python-performance-cython"
---

# Cython

## Introduction

Cython is a programming language that is a superset of Python, adding optional static type declarations that allow it to compile to highly optimized C code. It bridges the gap between Python's ease of use and C's raw performance, making it the most popular tool for writing Python extensions.

## Key Concepts

- **cdef declarations**: Cython's syntax for declaring C-typed variables (`cdef int x`), C functions (`cdef double compute()`), and C structs, enabling the compiler to generate fast C code instead of slow Python object operations.
- **Typed memoryviews**: Cython's mechanism for efficient, direct access to memory buffers (NumPy arrays, bytes) without Python overhead, using syntax like `double[:, :] arr`.
- **cpdef functions**: Functions declared with `cpdef` are callable from both Python and C code, providing fast C-level calls internally while remaining accessible from Python.
- **Extension types (cdef class)**: Cython's equivalent of `__slots__` classes, implemented as C structs with direct attribute access instead of dictionary lookups.

## Real World Context

Cython is used by some of the most performance-critical Python libraries: scikit-learn's machine learning algorithms, scipy's scientific computing routines, and lxml's XML parsing are all written in Cython. When you call `sklearn.fit()`, Cython-compiled inner loops process your data at near-C speed while maintaining a pure Python API.

## Deep Dive

### Basic Cython: Adding Types

The simplest Cython optimization is adding type declarations to Python code. Consider a pure Python function:

```python
# slow_python.py
def sum_squares(n):
    total = 0
    for i in range(n):
        total += i * i
    return total
```

The Cython version adds type information:

```cython
# fast_cython.pyx
def sum_squares(int n):
    cdef long total = 0
    cdef int i
    for i in range(n):
        total += i * i
    return total
```

This small change can yield 100x speedups because the compiler knows `i` and `total` are integers and can use CPU integer instructions directly instead of Python object operations.

### Building With setup.py

Cython files (.pyx) must be compiled before use:

```python
# setup.py
from setuptools import setup
from Cython.Build import cythonize

setup(
    ext_modules=cythonize("fast_cython.pyx"),
)
```

```bash
# Build the extension
python setup.py build_ext --inplace

# Then import like a regular module
# from fast_cython import sum_squares
```

### cdef Functions: C-Only Speed

```cython
# math_helpers.pyx

# cdef = C-only function (fastest, not callable from Python)
cdef double c_distance(double x1, double y1, double x2, double y2):
    cdef double dx = x2 - x1
    cdef double dy = y2 - y1
    return (dx * dx + dy * dy) ** 0.5

# cpdef = callable from both C and Python
cpdef double distance(double x1, double y1, double x2, double y2):
    return c_distance(x1, y1, x2, y2)

# def = normal Python function (slowest)
def py_distance(x1, y1, x2, y2):
    return distance(x1, y1, x2, y2)
```

### Typed Memoryviews for NumPy

Memoryviews provide direct, fast access to NumPy array data:

```cython
# matrix_ops.pyx
import numpy as np
cimport numpy as np

def row_sums(double[:, :] matrix):
    cdef int rows = matrix.shape[0]
    cdef int cols = matrix.shape[1]
    cdef double[:] result = np.zeros(rows)
    cdef int i, j

    for i in range(rows):
        for j in range(cols):
            result[i] += matrix[i, j]

    return np.asarray(result)
```

This accesses array elements directly through pointers, avoiding Python object creation for each element.

### Extension Types

```cython
# particle.pyx
cdef class Particle:
    cdef public double x, y, z
    cdef public double vx, vy, vz
    cdef double mass

    def __init__(self, double x, double y, double z, double mass):
        self.x = x
        self.y = y
        self.z = z
        self.mass = mass

    cpdef void update(self, double dt):
        self.x += self.vx * dt
        self.y += self.vy * dt
        self.z += self.vz * dt
```

## Common Pitfalls

1. **Forgetting to type loop variables** -- If the loop variable `i` in `for i in range(n)` is untyped, Cython creates a Python int object on every iteration, negating most performance gains.
2. **Using Python objects in inner loops** -- Calling Python methods (like `list.append()`) inside a typed Cython loop forces a transition back to Python, creating a bottleneck that dominates the loop's runtime.

## Best Practices

1. **Use `cython -a` to generate HTML annotation** -- The annotation report highlights yellow lines that still use Python operations, showing you exactly where to add type declarations for the biggest speedups.
2. **Start with `cpdef` for public functions** -- `cpdef` functions are fast when called from Cython code and still accessible from Python, giving you the best of both worlds.
3. **Use typed memoryviews instead of NumPy indexing** -- `double[:, :] arr` gives direct memory access, avoiding Python object creation on each array element access.

## Summary

- Cython compiles Python-like code to C, with type declarations enabling 10-100x speedups.
- `cdef` variables and functions bypass Python's object system for C-level speed.
- Typed memoryviews provide direct, zero-overhead access to NumPy array data.
- Use `cython -a` to generate annotation reports showing which lines still use slow Python operations.
- Build with `cythonize()` in setup.py, then import the compiled module like regular Python.

## Code Examples

**Complete setup.py for building Cython extensions with NumPy support and performance-oriented compiler directives**

```python
# setup.py for building Cython extensions
from setuptools import setup, Extension
from Cython.Build import cythonize
import numpy as np

extensions = [
    Extension(
        "fast_math",
        sources=["fast_math.pyx"],
        include_dirs=[np.get_include()],
    )
]

setup(
    name="fast_math",
    ext_modules=cythonize(
        extensions,
        compiler_directives={
            'boundscheck': False,   # Disable bounds checking
            'wraparound': False,    # Disable negative indexing
            'cdivision': True,      # Use C division (no ZeroDivisionError)
        },
    ),
)

# Build: python setup.py build_ext --inplace
# Then: from fast_math import sum_squares
```

**Demonstrating the performance gap that Cython bridges: pure Python loops vs optimized approaches, where Cython with type declarations achieves near-C speed**

```python
# Comparing Python vs Cython-style approaches
# (This is pure Python simulating what Cython does)
import time
import array

def sum_squares_python(n):
    """Pure Python: creates int objects each iteration."""
    total = 0
    for i in range(n):
        total += i * i
    return total

def sum_squares_optimized(n):
    """Optimized Python: generator expression with C-speed sum()."""
    return sum(i * i for i in range(n))

N = 10_000_000

start = time.perf_counter()
sum_squares_python(N)
python_time = time.perf_counter() - start

start = time.perf_counter()
sum_squares_optimized(N)
optimized_time = time.perf_counter() - start

print(f"Python loop:  {python_time*1000:.1f}ms")
print(f"Optimized:    {optimized_time*1000:.1f}ms")
# Cython with cdef types would be ~100x faster than the Python loop
```


## Resources

- [Cython Documentation](https://docs.python.org/3.14/extending/index.html) — Python 3.14 extending and embedding reference

---

> 📘 *This lesson is part of the [Python Performance: JIT & Internals](https://stanza.dev/courses/python-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*