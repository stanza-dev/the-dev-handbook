---
source_course: "python-performance"
source_lesson: "python-performance-string-optimization"
---

# String & Collection Optimization

## Introduction

String building and collection operations are among the most frequently written Python code, yet small choices in how you concatenate strings or check membership can create order-of-magnitude performance differences. This lesson covers the key patterns that separate O(n) from O(n^2) in everyday code.

## Key Concepts

- **`str.join()`**: Builds a string from an iterable in a single pass by pre-calculating the total size, achieving O(n) time versus O(n^2) for repeated `+=`.
- **Generator expression**: A lazy iterator that produces values one at a time, avoiding the memory cost of materializing an entire list.
- **Set membership testing**: Checking `item in set` is O(1) average case via hash lookup, versus O(n) for `item in list`.
- **`collections.deque`**: A double-ended queue with O(1) append and pop from both ends, unlike `list.pop(0)` which is O(n).

## Real World Context

Building HTML or CSV output from thousands of rows, filtering large datasets for duplicate detection, and implementing task queues or BFS algorithms all benefit directly from choosing the right string and collection operations.

## Deep Dive

### String Concatenation: join() vs +=

Repeated `+=` on strings creates a new string object on each iteration because strings are immutable. For n concatenations, this copies approximately 1 + 2 + 3 + ... + n characters, totaling O(n^2):

```python
# O(n^2) — each += allocates a new, larger string
result = ""
for word in words:
    result += word  # Copies all previous characters every time
```

`str.join()` pre-calculates the total length, allocates once, and copies each string exactly once:

```python
# O(n) — single allocation, one copy per string
result = "".join(words)
```

For building strings incrementally (e.g., in a loop with formatting), collect parts in a list, then join:

```python
parts = []
for item in items:
    parts.append(f"{item.name}: {item.value}")
result = "\n".join(parts)
```

Alternatively, `io.StringIO` provides a file-like buffer:

```python
from io import StringIO

buffer = StringIO()
for item in items:
    buffer.write(f"{item.name}: {item.value}\n")
result = buffer.getvalue()
```

### Generators vs Lists

When you only need to iterate once, a generator avoids allocating the entire result in memory:

```python
# Materializes a list of 10 million items in memory
total = sum([x ** 2 for x in range(10_000_000)])  # ~80 MB

# Generates values one at a time, constant memory
total = sum(x ** 2 for x in range(10_000_000))    # ~0 MB extra
```

### Set for Membership Testing

Checking `item in list` scans every element. Converting to a set gives O(1) lookups:

```python
# O(n) per lookup, O(n*m) total for m lookups
blacklist = ["spam", "phishing", "malware", ...]  # 100,000 entries
for email in emails:
    if email.sender in blacklist:  # Linear scan each time
        flag(email)

# O(1) per lookup, O(m) total after O(n) set construction
blacklist_set = set(blacklist)
for email in emails:
    if email.sender in blacklist_set:  # Hash lookup
        flag(email)
```

### deque for Queue Operations

`list.pop(0)` shifts every element, making it O(n). `collections.deque` uses a doubly-linked block structure:

```python
from collections import deque

# O(n) per pop from front — shifts all elements
queue = [1, 2, 3, 4, 5]
queue.pop(0)  # Slow for large lists

# O(1) per pop from either end
queue = deque([1, 2, 3, 4, 5])
queue.popleft()  # Fast regardless of size
```

## Common Pitfalls

- **Using `+=` in a tight loop for string building**: This is the single most common Python performance mistake. Always use `join()` or `StringIO` for building strings from many pieces.
- **Converting to a set for a single membership check**: If you only check membership once, the O(n) cost of building the set exceeds the O(n) cost of a single linear scan. Sets pay off when you perform multiple lookups.
- **Using a list comprehension inside `sum()`, `min()`, `max()`**: These functions accept generators directly. Wrapping in `[...]` wastes memory for no benefit.

## Best Practices

- Default to `str.join()` for combining multiple strings. Reserve `+=` for 2-3 concatenations where readability wins.
- Use generator expressions instead of list comprehensions when the result is consumed once (passed to `sum()`, `any()`, `all()`, etc.).
- Convert lookup collections to `set` or `frozenset` upfront when performing repeated membership tests.

## Summary

- `str.join()` builds strings in O(n) time; repeated `+=` is O(n^2) because each concatenation copies all previous characters.
- Generator expressions produce values lazily, avoiding the memory cost of materializing full lists.
- `set` provides O(1) average membership testing versus O(n) for lists — convert once, query many times.
- `collections.deque` gives O(1) popleft/appendleft, making it the correct choice for FIFO queues.

## Code Examples

**Benchmarking three string concatenation strategies: += loop, str.join(), and StringIO**

```python
import timeit

words = ["hello"] * 50_000

def concat_plus():
    result = ""
    for w in words:
        result += w
    return result

def concat_join():
    return "".join(words)

def concat_stringio():
    from io import StringIO
    buf = StringIO()
    for w in words:
        buf.write(w)
    return buf.getvalue()

print(f"+=       : {timeit.timeit(concat_plus, number=10):.4f}s")
print(f"join()   : {timeit.timeit(concat_join, number=10):.4f}s")
print(f"StringIO : {timeit.timeit(concat_stringio, number=10):.4f}s")
```

**Comparing list.pop(0) versus deque.popleft() for FIFO queue draining**

```python
import timeit
from collections import deque

def list_queue(n=10_000):
    q = list(range(n))
    while q:
        q.pop(0)  # O(n) shift each time

def deque_queue(n=10_000):
    q = deque(range(n))
    while q:
        q.popleft()  # O(1) each time

print(f"list.pop(0)   : {timeit.timeit(list_queue, number=10):.4f}s")
print(f"deque.popleft(): {timeit.timeit(deque_queue, number=10):.4f}s")
```


## Resources

- [str.join — Built-in Types](https://docs.python.org/3.14/library/stdtypes.html#str.join) — Official documentation for the str.join() method
- [collections.deque](https://docs.python.org/3.14/library/collections.html#collections.deque) — Official documentation for deque including performance characteristics

---

> 📘 *This lesson is part of the [Python Performance: JIT & Internals](https://stanza.dev/courses/python-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*