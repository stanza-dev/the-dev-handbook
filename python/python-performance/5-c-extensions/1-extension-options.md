---
source_course: "python-performance"
source_lesson: "python-performance-extension-options"
---

# When to Use Extensions

## Decision Tree

```
Need better performance?
├─ Try PyPy first (JIT-compiled Python)
├─ Use NumPy/Pandas for numerical work
├─ Profile to find bottleneck
└─ If still slow → Consider extensions
```

## Extension Technologies

| Technology | Language | Learning Curve | Safety | Speed |
|------------|----------|----------------|--------|-------|
| C API | C | High | Low | Fastest |
| Cython | Python-like | Medium | Medium | Very Fast |
| PyO3 | Rust | High | High | Very Fast |
| cffi | Python + C | Low | Medium | Fast |
| ctypes | Python | Low | Low | Fast |

## Quick Wins First

```python
# Before writing C, try:

# 1. NumPy for arrays
import numpy as np
arr = np.array([1, 2, 3])
result = np.sum(arr ** 2)  # Runs in C

# 2. Built-in functions
sum(iterable)  # Faster than Python loop
map(func, items)  # C-speed iteration

# 3. List comprehensions
[x**2 for x in range(1000)]  # Optimized
```

## Code Examples

**NumPy vs pure Python**

```python
# Performance comparison example
import time
import numpy as np

def python_sum_squares(n):
    return sum(i**2 for i in range(n))

def numpy_sum_squares(n):
    arr = np.arange(n)
    return np.sum(arr ** 2)

n = 1_000_000

start = time.perf_counter()
python_sum_squares(n)
print(f"Python: {time.perf_counter() - start:.4f}s")

start = time.perf_counter()
numpy_sum_squares(n)
print(f"NumPy: {time.perf_counter() - start:.4f}s")
```


---

> 📘 *This lesson is part of the [Python Performance: JIT & Internals](https://stanza.dev/courses/python-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*