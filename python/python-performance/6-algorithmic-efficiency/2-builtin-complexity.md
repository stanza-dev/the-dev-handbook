---
source_course: "python-performance"
source_lesson: "python-performance-builtin-complexity"
---

# Python Data Structure Complexity

## List Operations

| Operation | Complexity | Notes |
|-----------|------------|-------|
| `list[i]` | O(1) | Index access |
| `list.append(x)` | O(1) | Amortized |
| `list.pop()` | O(1) | From end |
| `list.pop(0)` | O(n) | Shifts all elements |
| `list.insert(0, x)` | O(n) | Shifts all elements |
| `x in list` | O(n) | Linear search |
| `list.sort()` | O(n log n) | Timsort |

## Dict Operations

| Operation | Average | Worst |
|-----------|---------|-------|
| `dict[key]` | O(1) | O(n) |
| `dict[key] = val` | O(1) | O(n) |
| `key in dict` | O(1) | O(n) |
| `del dict[key]` | O(1) | O(n) |

Worst case happens with hash collisions.

## Set Operations

| Operation | Complexity |
|-----------|------------|
| `x in set` | O(1) average |
| `set.add(x)` | O(1) average |
| `set1 & set2` | O(min(n, m)) |
| `set1 \| set2` | O(n + m) |

## Deque Operations

```python
from collections import deque

dq = deque()
dq.append(x)      # O(1)
dq.appendleft(x)  # O(1)
dq.pop()          # O(1)
dq.popleft()      # O(1) - Unlike list!
dq[i]             # O(n) - Unlike list!
```

## Code Examples

**Choosing efficient data structures**

```python
# Choosing the right data structure

# Bad: Using list for queue
queue = []
queue.append(item)   # O(1)
queue.pop(0)         # O(n) - slow!

# Good: Using deque for queue
from collections import deque
queue = deque()
queue.append(item)   # O(1)
queue.popleft()      # O(1) - fast!

# Bad: Membership testing in list
if user_id in user_list:  # O(n)
    pass

# Good: Membership testing in set
if user_id in user_set:   # O(1)
    pass
```


---

> 📘 *This lesson is part of the [Python Performance: JIT & Internals](https://stanza.dev/courses/python-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*