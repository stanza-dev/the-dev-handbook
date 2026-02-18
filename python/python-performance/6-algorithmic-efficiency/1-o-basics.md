---
source_course: "python-performance"
source_lesson: "python-performance-big-o-basics"
---

# Big-O Notation

Big-O describes how algorithm performance scales with input size.

## Common Complexities

| Big-O | Name | Example |
|-------|------|--------|
| O(1) | Constant | Dict lookup |
| O(log n) | Logarithmic | Binary search |
| O(n) | Linear | List iteration |
| O(n log n) | Linearithmic | Sorting |
| O(n²) | Quadratic | Nested loops |
| O(2ⁿ) | Exponential | Subsets |

## Practical Impact

```
n = 1,000,000

O(1):       1 operation
O(log n):   20 operations
O(n):       1,000,000 operations
O(n log n): 20,000,000 operations
O(n²):      1,000,000,000,000 operations!
```

## Identifying Complexity

```python
# O(1) - Direct access
value = my_list[0]

# O(n) - Single loop
for item in items:
    process(item)

# O(n²) - Nested loops
for i in items:
    for j in items:
        compare(i, j)

# O(log n) - Halving each step
while n > 0:
    n //= 2
```

## Code Examples

**Measuring algorithm scaling**

```python
import time

def measure_scaling(func, sizes):
    """Measure how function scales with input size."""
    for n in sizes:
        data = list(range(n))
        start = time.perf_counter()
        func(data)
        elapsed = time.perf_counter() - start
        print(f"n={n:>8}: {elapsed:.6f}s")

# Test list operations
def linear_search(data):
    return -1 in data  # O(n)

measure_scaling(linear_search, [1000, 10000, 100000, 1000000])
```


---

> 📘 *This lesson is part of the [Python Performance: JIT & Internals](https://stanza.dev/courses/python-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*