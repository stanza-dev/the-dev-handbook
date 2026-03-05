---
source_course: "python-performance"
source_lesson: "python-performance-big-o-basics"
---

# Big-O Notation

## Introduction

Big-O notation describes how an algorithm's runtime or memory usage grows as the input size increases. Understanding Big-O is essential for writing performant Python code because it tells you whether your solution will scale to real-world data sizes or collapse under load.

## Key Concepts

- **O(1) -- Constant time**: The operation takes the same time regardless of input size, like accessing a dictionary key or indexing a list.
- **O(log n) -- Logarithmic time**: The work halves with each step, like binary search. Doubling the input adds only one extra step.
- **O(n) -- Linear time**: The work grows proportionally with input size, like iterating through a list once.
- **O(n log n) -- Linearithmic time**: The complexity of efficient sorting algorithms like Timsort (Python's `sorted()`), combining a linear pass with logarithmic subdivisions.
- **O(n^2) -- Quadratic time**: Nested loops over the same data. Doubling the input quadruples the runtime, making it impractical for large datasets.
- **O(2^n) -- Exponential time**: Each additional input element doubles the work, like brute-force solutions to the traveling salesman problem. Only feasible for very small inputs.

## Real World Context

Algorithmic complexity determines whether your code handles production traffic or crashes. A function that takes 1ms for 100 items at O(n^2) will take 100ms for 1,000 items, 10 seconds for 10,000 items, and 17 minutes for 100,000 items. Choosing an O(n) algorithm instead means all four cases complete in under 100ms. This difference is why senior engineers focus on algorithmic efficiency before micro-optimizations.

## Deep Dive

### Complexity at Scale

Here is how different complexities compare as input grows:

| n | O(1) | O(log n) | O(n) | O(n log n) | O(n^2) | O(2^n) |
|---|------|----------|------|------------|--------|--------|
| 10 | 1 | 3 | 10 | 33 | 100 | 1,024 |
| 100 | 1 | 7 | 100 | 664 | 10,000 | 1.3 x 10^30 |
| 1,000 | 1 | 10 | 1,000 | 9,966 | 1,000,000 | -- |
| 1,000,000 | 1 | 20 | 1,000,000 | 19,931,569 | 10^12 | -- |

### Identifying Complexity From Code

Learn to recognize patterns:

```python
# O(1) - Dictionary lookup
def get_user(users_dict, user_id):
    return users_dict.get(user_id)  # Hash table lookup

# O(log n) - Binary search
def binary_search(sorted_list, target):
    low, high = 0, len(sorted_list) - 1
    while low <= high:
        mid = (low + high) // 2
        if sorted_list[mid] == target:
            return mid
        elif sorted_list[mid] < target:
            low = mid + 1
        else:
            high = mid - 1
    return -1

# O(n) - Single loop
def find_max(items):
    max_val = items[0]
    for item in items:
        if item > max_val:
            max_val = item
    return max_val

# O(n log n) - Sorting
def sort_and_find(items, k):
    sorted_items = sorted(items)  # O(n log n)
    return sorted_items[k]        # O(1)

# O(n^2) - Nested loops
def has_duplicate_naive(items):
    for i in range(len(items)):
        for j in range(i + 1, len(items)):
            if items[i] == items[j]:
                return True
    return False

# O(n) - Same problem, better algorithm
def has_duplicate_fast(items):
    seen = set()
    for item in items:
        if item in seen:
            return True
        seen.add(item)
    return False
```

### Rules of Thumb

1. **Drop constants**: O(2n) = O(n), O(n/2) = O(n)
2. **Drop lower-order terms**: O(n^2 + n) = O(n^2)
3. **Multiplication rule**: Nested loops multiply: O(n) * O(m) = O(nm)
4. **Addition rule**: Sequential steps add: O(n) + O(m) = O(n + m)

### Practical Impact

```python
import time

def measure_scaling(func, sizes):
    for n in sizes:
        data = list(range(n))
        start = time.perf_counter()
        func(data)
        elapsed = time.perf_counter() - start
        print(f"  n={n:>8}: {elapsed*1000:>10.2f}ms")

print("O(n) - Linear scan:")
measure_scaling(lambda d: sum(d), [1000, 10000, 100000])

print("O(n^2) - Nested loop:")
def nested(d):
    for i in d:
        for j in d:
            pass
measure_scaling(nested, [1000, 3000, 5000])
```

## Common Pitfalls

1. **Hidden O(n) operations in loops** -- Using `item in list` inside a loop creates O(n^2) complexity because list membership testing is O(n). Replacing the list with a set makes the inner check O(1), reducing overall complexity to O(n).
2. **Confusing average and worst case** -- Dictionary operations are O(1) on average but O(n) in the worst case (hash collisions). For most practical purposes average case matters, but be aware of pathological inputs.
3. **Ignoring the constant factor** -- O(n) with a large constant (e.g., 1000n) can be slower than O(n^2) for small inputs. Big-O describes growth rate, not absolute speed.

## Best Practices

1. **Think about complexity before coding** -- Sketch the algorithm and estimate its Big-O before writing code. Refactoring an O(n^2) solution later is much harder than designing an O(n) solution upfront.
2. **Use Python's built-in data structures** -- Python's dict, set, and deque are highly optimized C implementations. Reach for them before implementing custom data structures.
3. **Benchmark at realistic scale** -- Test with production-sized data, not toy examples. An O(n^2) algorithm that works fine with 100 items may be unusable with 100,000.

## Summary

- Big-O notation describes how runtime scales with input size, not absolute speed.
- O(1) and O(log n) scale well; O(n^2) and worse become impractical for large inputs.
- Learn to recognize complexity patterns: single loops are O(n), nested loops are O(n^2), and divide-and-conquer is O(n log n).
- Converting O(n^2) to O(n) using hash sets is one of the most common and impactful optimizations.
- Always benchmark at realistic data sizes to catch scaling problems before production.

## Code Examples

**Side-by-side comparison of O(n^2) vs O(n) duplicate detection, showing how the performance gap widens dramatically as input size grows**

```python
import time

def find_duplicates_quadratic(items):
    """O(n^2): Compare every pair."""
    duplicates = []
    for i in range(len(items)):
        for j in range(i + 1, len(items)):
            if items[i] == items[j] and items[i] not in duplicates:
                duplicates.append(items[i])
    return duplicates

def find_duplicates_linear(items):
    """O(n): Use a set to track seen items."""
    seen = set()
    duplicates = set()
    for item in items:
        if item in seen:
            duplicates.add(item)
        seen.add(item)
    return list(duplicates)

# Benchmark at different scales
for n in [1_000, 5_000, 10_000]:
    data = list(range(n)) + list(range(n // 2))  # Some duplicates

    start = time.perf_counter()
    find_duplicates_quadratic(data)
    quad_time = time.perf_counter() - start

    start = time.perf_counter()
    find_duplicates_linear(data)
    linear_time = time.perf_counter() - start

    print(f"n={n:>6}: O(n^2)={quad_time*1000:>8.2f}ms  "
          f"O(n)={linear_time*1000:>6.2f}ms  "
          f"Speedup={quad_time/max(linear_time, 1e-9):>.0f}x")
```


## Resources

- [TimeComplexity - Python Wiki](https://docs.python.org/3.14/faq/design.html) — Python design FAQ covering performance characteristics of built-in types
- [Data Structures](https://docs.python.org/3.14/tutorial/datastructures.html) — Python 3.14 tutorial on lists, tuples, sets, and dictionaries

---

> 📘 *This lesson is part of the [Python Performance: JIT & Internals](https://stanza.dev/courses/python-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*