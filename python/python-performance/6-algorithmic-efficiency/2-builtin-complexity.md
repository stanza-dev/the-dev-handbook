---
source_course: "python-performance"
source_lesson: "python-performance-builtin-complexity"
---

# Built-in Type Complexity

## Introduction

Python's built-in data structures -- list, dict, set, tuple, and deque -- each have different performance characteristics for different operations. Knowing these complexities lets you choose the right structure for your use case and avoid accidental O(n) operations hidden inside loops.

## Key Concepts

- **List**: A dynamic array. Fast O(1) append and index access, but O(n) for insert at the beginning and membership testing (`in` operator).
- **Dict**: A hash table. O(1) average for get, set, and delete operations. The most versatile high-performance structure in Python.
- **Set**: A hash table without values. O(1) average for add, remove, and membership testing. Essential for fast lookups and deduplication.
- **Deque**: A double-ended queue (from `collections`). O(1) append and pop from both ends, but O(n) for random index access.
- **Amortized complexity**: Some operations (like `list.append`) are O(1) most of the time but occasionally O(n) when the underlying array must be resized. Averaged over many operations, the cost is still O(1).

## Real World Context

Choosing the wrong data structure is one of the most common performance mistakes in Python. A real-world example: checking if a user ID exists in a list of 100,000 IDs inside a request handler. With a list, each check is O(n), taking ~5ms. Converting to a set makes it O(1), taking ~0.00005ms. For an endpoint handling 1,000 requests/second, this is the difference between an overloaded server and one barely breaking a sweat.

## Deep Dive

### List Operations

```python
my_list = [1, 2, 3, 4, 5]

# O(1) operations
my_list[2]              # Index access
my_list.append(6)       # Append to end (amortized)
my_list.pop()           # Pop from end
len(my_list)            # Length

# O(n) operations -- AVOID IN LOOPS
my_list.insert(0, 0)    # Insert at beginning (shifts all elements)
my_list.pop(0)          # Pop from beginning (shifts all elements)
3 in my_list            # Membership test (linear scan)
my_list.remove(3)       # Remove by value (linear scan)
my_list.index(3)        # Find index (linear scan)
```

### Dict Operations

```python
my_dict = {'a': 1, 'b': 2, 'c': 3}

# O(1) average operations
my_dict['a']            # Get by key
my_dict['d'] = 4        # Set key
del my_dict['a']        # Delete key
'b' in my_dict          # Membership test (checks keys)
my_dict.get('x', None)  # Get with default

# O(n) operations
list(my_dict.values())  # All values
list(my_dict.items())   # All key-value pairs
2 in my_dict.values()   # Value membership test
```

### Set Operations

```python
my_set = {1, 2, 3, 4, 5}
other_set = {4, 5, 6, 7}

# O(1) average operations
my_set.add(6)           # Add element
my_set.discard(1)       # Remove (no error if missing)
3 in my_set             # Membership test

# O(min(len(s1), len(s2))) for set operations
my_set & other_set      # Intersection: {4, 5}
my_set | other_set      # Union: {1, 2, 3, 4, 5, 6, 7}
my_set - other_set      # Difference: {1, 2, 3}
my_set ^ other_set      # Symmetric difference: {1, 2, 3, 6, 7}
```

### Deque Operations

```python
from collections import deque

dq = deque([1, 2, 3, 4, 5])

# O(1) operations (both ends!)
dq.append(6)            # Add to right
dq.appendleft(0)        # Add to left
dq.pop()                # Remove from right
dq.popleft()            # Remove from left

# O(n) operations
dq[2]                   # Random access (index)
3 in dq                 # Membership test
```

### Quick Reference Table

| Operation | list | dict | set | deque |
|-----------|------|------|-----|-------|
| Index `[i]` | O(1) | O(1) key | -- | O(n) |
| Append end | O(1)* | -- | -- | O(1) |
| Append start | O(n) | -- | -- | O(1) |
| `in` check | O(n) | O(1) | O(1) | O(n) |
| Delete by index | O(n) | O(1) key | O(1) | O(n) |
| Sort | O(n log n) | -- | -- | -- |

*Amortized O(1)

### The list.insert(0) Trap

A common performance mistake:

```python
import time
from collections import deque

# Building a sequence by prepending
n = 100_000

# SLOW: O(n^2) total -- each insert shifts all elements
start = time.perf_counter()
result = []
for i in range(n):
    result.insert(0, i)  # O(n) each time!
list_time = time.perf_counter() - start

# FAST: O(n) total -- O(1) each appendleft
start = time.perf_counter()
result = deque()
for i in range(n):
    result.appendleft(i)  # O(1) each time
deque_time = time.perf_counter() - start

print(f"list.insert(0):   {list_time*1000:.0f}ms")
print(f"deque.appendleft: {deque_time*1000:.0f}ms")
```

## Common Pitfalls

1. **Using `in` on a list inside a loop** -- Checking `item in large_list` is O(n). Inside another O(n) loop, this creates O(n^2) behavior. Convert the list to a set first for O(1) lookups.
2. **Using list.insert(0) or list.pop(0) repeatedly** -- Both are O(n) because they shift all subsequent elements. Use `collections.deque` for O(1) operations at both ends.
3. **Assuming deque is always better than list** -- Deque has O(n) random access (`dq[i]`), so if you need frequent indexing, a list is the better choice.

## Best Practices

1. **Convert lookup lists to sets** -- If you check membership more than once, convert to a set: `lookup = set(items)`. The O(n) conversion cost pays for itself after just one O(1) lookup.
2. **Use deque for queue/stack patterns** -- When you need to add or remove from both ends, `collections.deque` provides O(1) at both ends versus list's O(n) at the front.
3. **Profile before assuming** -- Python's built-in types are heavily optimized in C. A list with 100 elements may be faster than a set due to lower overhead, even for membership tests. Profile at your actual data size.

## Summary

- Lists are O(1) for append and index access, but O(n) for insert(0), pop(0), and `in` checks.
- Dicts and sets provide O(1) average lookup, insertion, and deletion via hash tables.
- Deque gives O(1) at both ends but O(n) for index access -- use it for queues, not random access.
- The single most impactful optimization is often converting a list to a set for membership testing.
- Always consider the complexity of operations inside loops to avoid accidental O(n^2) behavior.

## Code Examples

**Demonstrating the dramatic performance difference between O(n) list membership checks and O(1) set membership checks in a filtering operation**

```python
import time

# The list-to-set optimization: most impactful single change
users = list(range(100_000))
blocked_list = list(range(0, 100_000, 3))  # 33k blocked users
blocked_set = set(blocked_list)            # Same data, O(1) lookup

# O(n * m) with list: check each user against blocked list
start = time.perf_counter()
allowed_list = [u for u in users if u not in blocked_list]
list_time = time.perf_counter() - start

# O(n) with set: check each user against blocked set
start = time.perf_counter()
allowed_set = [u for u in users if u not in blocked_set]
set_time = time.perf_counter() - start

print(f"List lookup: {list_time*1000:.0f}ms")
print(f"Set lookup:  {set_time*1000:.0f}ms")
print(f"Speedup:     {list_time/set_time:.0f}x")
print(f"Same result: {allowed_list == allowed_set}")
```


## Resources

- [collections -- Container datatypes](https://docs.python.org/3.14/library/collections.html) — Official documentation for deque, Counter, defaultdict and more
- [Built-in Types](https://docs.python.org/3.14/library/stdtypes.html) — Python 3.14 reference for list, dict, set, and other built-in types

---

> 📘 *This lesson is part of the [Python Performance: JIT & Internals](https://stanza.dev/courses/python-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*