---
source_course: "python-performance"
source_lesson: "python-performance-algorithmic-patterns"
---

# Common Optimization Patterns

## Introduction

Most algorithmic optimizations follow a handful of recurring patterns. Learning these patterns lets you recognize O(n^2) code and transform it to O(n) or O(n log n) systematically, rather than through trial and error. These patterns apply across all programming languages but are especially impactful in Python where each operation has more overhead.

## Key Concepts

- **Hash set conversion**: Replacing nested loops with a set lookup to convert O(n^2) to O(n) by trading space for time.
- **Precomputation**: Computing intermediate results once and storing them (in a dict or list) to avoid redundant recalculation inside loops.
- **Early exit**: Using Python's `any()`, `all()`, or explicit `break`/`return` to stop processing as soon as the answer is known.
- **Built-in C-speed functions**: Leveraging `sum()`, `min()`, `max()`, `sorted()`, `map()`, and `filter()` which run their inner loops in C rather than Python bytecode.

## Real World Context

In production systems, the difference between O(n^2) and O(n) is often the difference between a sub-second response and a timeout. An e-commerce search that checks each of 10,000 products against 5,000 filter criteria with nested loops makes 50 million comparisons. Converting filter criteria to a set reduces this to 10,000 set lookups -- a 5,000x improvement that brings the response from 30 seconds to 6 milliseconds.

## Deep Dive

### Pattern 1: Replace Nested Loops with Hash Sets

The most common optimization -- converting O(n^2) to O(n):

```python
# BEFORE: O(n * m) -- nested loop
def find_common_naive(list_a, list_b):
    common = []
    for item in list_a:       # O(n)
        if item in list_b:    # O(m) -- linear scan!
            common.append(item)
    return common

# AFTER: O(n + m) -- hash set
def find_common_fast(list_a, list_b):
    set_b = set(list_b)       # O(m) one-time cost
    return [item for item in list_a if item in set_b]  # O(n)
```

### Pattern 2: Precompute Results

Avoid recomputing the same values:

```python
# BEFORE: O(n * k) -- recomputes aggregates each iteration
def find_above_average_naive(groups):
    results = []
    for group in groups:
        avg = sum(group) / len(group)  # Recomputed each time
        results.append([x for x in group if x > avg])
    return results

# AFTER: O(n) -- precompute all averages
def find_above_average_fast(groups):
    averages = [sum(g) / len(g) for g in groups]  # Precompute
    return [
        [x for x in group if x > avg]
        for group, avg in zip(groups, averages)
    ]
```

Precomputation with dictionaries:

```python
# BEFORE: O(n^2) -- scan for each lookup
def process_orders(orders, products):
    for order in orders:
        for product in products:           # O(n) scan
            if product['id'] == order['product_id']:
                order['name'] = product['name']
                break

# AFTER: O(n) -- build lookup dict
def process_orders_fast(orders, products):
    product_map = {p['id']: p for p in products}  # O(n) once
    for order in orders:
        product = product_map.get(order['product_id'])  # O(1)
        if product:
            order['name'] = product['name']
```

### Pattern 3: Early Exit

Stop as soon as you have the answer:

```python
# SLOW: Processes ALL items even if answer found early
def has_negative_naive(items):
    negatives = [x for x in items if x < 0]
    return len(negatives) > 0

# FAST: Stops at first negative (short-circuit)
def has_negative_fast(items):
    return any(x < 0 for x in items)

# all() also short-circuits
def all_positive(items):
    return all(x > 0 for x in items)  # Stops at first non-positive
```

### Pattern 4: Use Built-in C-Speed Functions

Python's built-ins run their loops in C:

```python
import time

numbers = list(range(1_000_000))

# Python loop: ~50ms
start = time.perf_counter()
total = 0
for n in numbers:
    total += n
python_time = time.perf_counter() - start

# Built-in sum(): ~5ms (10x faster)
start = time.perf_counter()
total = sum(numbers)
builtin_time = time.perf_counter() - start

print(f"Python loop: {python_time*1000:.1f}ms")
print(f"sum():       {builtin_time*1000:.1f}ms")
```

Other fast built-ins:

```python
# Instead of manual loops, use:
max_val = max(items)              # C-speed max
min_val = min(items)              # C-speed min
sorted_items = sorted(items)      # C-speed Timsort
mapped = list(map(str, items))    # C-speed mapping
filtered = list(filter(None, items))  # C-speed filtering
```

### Combining Patterns

```python
# Real-world example: Find users with duplicate emails
# across multiple organizations

# BEFORE: O(n^2) -- compare every pair
def find_duplicate_emails_naive(orgs):
    all_users = [u for org in orgs for u in org['users']]
    duplicates = []
    for i, u1 in enumerate(all_users):
        for u2 in all_users[i+1:]:
            if u1['email'] == u2['email']:
                duplicates.append(u1['email'])
    return list(set(duplicates))

# AFTER: O(n) -- hash set + precomputation
def find_duplicate_emails_fast(orgs):
    seen = set()
    duplicates = set()
    for org in orgs:
        for user in org['users']:
            email = user['email']
            if email in seen:      # O(1) lookup
                duplicates.add(email)
            seen.add(email)
    return list(duplicates)
```

## Common Pitfalls

1. **Over-engineering simple cases** -- If your list has 50 items, an O(n^2) algorithm takes 2,500 operations and completes in microseconds. The overhead of creating a set may not be worth it for very small inputs.
2. **Forgetting that `any()` and `all()` need generator expressions** -- Using `any([x > 0 for x in items])` creates the entire list first, defeating the short-circuit benefit. Use `any(x > 0 for x in items)` without brackets.
3. **Premature optimization without profiling** -- Spending hours optimizing a function that takes 0.1% of total runtime provides negligible benefit. Profile first to find the real bottleneck.

## Best Practices

1. **Convert lookup collections to sets or dicts first** -- Whenever you check membership more than once, pay the O(n) conversion cost upfront to get O(1) lookups in the hot path.
2. **Use `any()` and `all()` with generators for early exit** -- These built-ins short-circuit and run in C, providing both algorithmic and constant-factor improvements.
3. **Prefer built-in functions over manual loops** -- `sum()`, `max()`, `min()`, `sorted()`, `map()`, and `filter()` execute their inner loops in C, typically 5-10x faster than equivalent Python loops.

## Summary

- The hash set pattern converts O(n^2) nested lookups to O(n) by trading space for time.
- Precomputing intermediate results into dicts avoids redundant O(n) scans inside loops.
- `any()` and `all()` with generator expressions provide early exit without processing the full collection.
- Python's built-in functions run their loops in C and are typically 5-10x faster than manual Python loops.
- These patterns can be combined: build a lookup dict, use it with early exit, and leverage built-ins for aggregation.

## Code Examples

**Classic two-sum problem demonstrating the hash map pattern: converting O(n^2) brute force to O(n) by storing complements in a dictionary**

```python
import time

# Pattern: Convert O(n^2) to O(n) with a hash set
def two_sum_quadratic(numbers, target):
    """O(n^2): Check every pair."""
    for i in range(len(numbers)):
        for j in range(i + 1, len(numbers)):
            if numbers[i] + numbers[j] == target:
                return (i, j)
    return None

def two_sum_linear(numbers, target):
    """O(n): Use a dict to find complements."""
    seen = {}  # value -> index
    for i, num in enumerate(numbers):
        complement = target - num
        if complement in seen:  # O(1) lookup
            return (seen[complement], i)
        seen[num] = i
    return None

# Benchmark
numbers = list(range(10_000))
target = 15_000  # Forces near-worst case

start = time.perf_counter()
two_sum_quadratic(numbers, target)
quad_time = time.perf_counter() - start

start = time.perf_counter()
two_sum_linear(numbers, target)
linear_time = time.perf_counter() - start

print(f"O(n^2): {quad_time*1000:.2f}ms")
print(f"O(n):   {linear_time*1000:.2f}ms")
print(f"Speedup: {quad_time/max(linear_time, 1e-9):.0f}x")
```

**Demonstrating the massive performance difference between any() with a list comprehension (processes everything) versus a generator expression (short-circuits at first match)**

```python
# Early exit with any() and all()
import time

data = list(range(1_000_000))

# SLOW: Creates the full list, then checks
start = time.perf_counter()
result = any([x > 5 for x in data])  # List comprehension: processes ALL
list_time = time.perf_counter() - start

# FAST: Generator stops at first match
start = time.perf_counter()
result = any(x > 5 for x in data)   # Generator: stops after 6 items!
gen_time = time.perf_counter() - start

print(f"List comprehension: {list_time*1000:.2f}ms (processes all 1M items)")
print(f"Generator expr:     {gen_time*1000:.4f}ms (stops after ~6 items)")
print(f"Speedup: {list_time/max(gen_time, 1e-9):.0f}x")
```


## Resources

- [Functional Programming HOWTO](https://docs.python.org/3.14/howto/functional.html) — Guide covering map(), filter(), reduce() and other functional tools for efficient data processing
- [Built-in Functions](https://docs.python.org/3.14/library/functions.html) — Complete reference for Python built-in functions including sum(), any(), all(), sorted()

---

> 📘 *This lesson is part of the [Python Performance: JIT & Internals](https://stanza.dev/courses/python-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*