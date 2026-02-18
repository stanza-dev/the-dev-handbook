---
source_course: "python-performance"
source_lesson: "python-performance-cython"
---

# Cython: Python-like Syntax, C Speed

## What is Cython?

Cython compiles Python-like code to C, allowing:
- Static type declarations
- Direct C function calls
- 10-100x speedups for numerical code

## Basic Example

```python
# example.pyx
def python_loop(n):
    total = 0
    for i in range(n):
        total += i
    return total

# With type annotations
def cython_loop(int n):
    cdef int total = 0
    cdef int i
    for i in range(n):
        total += i
    return total
```

## Type Declarations

```python
# Variables
cdef int x = 10
cdef double y = 3.14
cdef list my_list = []

# Functions
cdef int fast_func(int a, int b):
    return a + b

# Typed memoryviews (for arrays)
def process(double[:] arr):  # 1D array
    cdef int i
    for i in range(arr.shape[0]):
        arr[i] *= 2
```

## Building

```python
# setup.py
from setuptools import setup
from Cython.Build import cythonize

setup(
    ext_modules=cythonize("example.pyx")
)
```

```bash
python setup.py build_ext --inplace
```

## NumPy Integration

```python
import numpy as np
cimport numpy as np

def fast_sum(np.ndarray[np.float64_t, ndim=1] arr):
    cdef double total = 0
    cdef int i
    for i in range(arr.shape[0]):
        total += arr[i]
    return total
```

## Code Examples

**Cython in notebooks**

```python
# Using Cython in Jupyter/IPython
# %load_ext Cython

# %%cython
# cdef int fibonacci(int n):
#     cdef int a = 0, b = 1
#     cdef int i
#     for i in range(n):
#         a, b = b, a + b
#     return a

# print(fibonacci(100))

# Pure Python equivalent for comparison
def py_fibonacci(n):
    a, b = 0, 1
    for _ in range(n):
        a, b = b, a + b
    return a
```


---

> 📘 *This lesson is part of the [Python Performance: JIT & Internals](https://stanza.dev/courses/python-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*