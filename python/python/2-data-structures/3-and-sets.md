---
source_course: "python"
source_lesson: "python-dictionaries-and-sets"
---

# Dictionaries and Sets

## Introduction
Dictionaries and sets are hash-based collections that give you O(1) average-time lookups. This lesson covers how to create, modify, and combine them, including the modern merge operators introduced in Python 3.9.

## Key Concepts
- **dict**: A mutable mapping of keys to values with O(1) average lookup.
- **set**: An unordered collection of unique, hashable elements.
- **Dictionary views**: Live objects (`keys()`, `values()`, `items()`) that reflect changes to the underlying dict.
- **Union operator `|`**: Merges two dicts or two sets into a new collection (Python 3.9+ for dicts).

## Real World Context
Dictionaries are the backbone of Python programming. JSON parsing produces dicts, function keyword arguments are stored in dicts, and class attributes live in `__dict__`. Sets power fast membership tests -- checking if a user ID is in a blocklist, deduplicating log entries, or computing the intersection of permission groups.

## Deep Dive

### Creating Dictionaries

```python
# Literal syntax
user = {"name": "Alice", "age": 30}

# From sequences
keys = ["a", "b", "c"]
values = [1, 2, 3]
d = dict(zip(keys, values))  # {'a': 1, 'b': 2, 'c': 3}

# Default values
d = dict.fromkeys(['a', 'b', 'c'], 0)  # {'a': 0, 'b': 0, 'c': 0}
```

### Dictionary Operations

```python
d = {"a": 1, "b": 2}

# Access
d["a"]              # 1 (raises KeyError if missing)
d.get("c", 0)       # 0 (default if missing)

# Modification
d["c"] = 3          # Add or update
d.update({"d": 4})  # Merge another dict
d.setdefault("e", 5)  # Set only if key missing

# Removal
del d["a"]          # Remove key
value = d.pop("b")  # Remove and return value
d.clear()           # Remove all items
```

### Dictionary Views

```python
d = {"a": 1, "b": 2}

d.keys()    # dict_keys(['a', 'b'])
d.values()  # dict_values([1, 2])
d.items()   # dict_items([('a', 1), ('b', 2)])

# Iteration
for key, value in d.items():
    print(f"{key}: {value}")
```

### Dictionary Merging (3.9+)

```python
defaults = {"theme": "light", "lang": "en"}
user = {"theme": "dark"}

# Union operator (creates new dict)
config = defaults | user  # {'theme': 'dark', 'lang': 'en'}

# In-place update
defaults |= user
```

### Sets

Sets are unordered collections of unique elements.

```python
a = {1, 2, 3}
b = {2, 3, 4}

a | b   # Union: {1, 2, 3, 4}
a & b   # Intersection: {2, 3}
a - b   # Difference: {1}
a ^ b   # Symmetric difference: {1, 4}

a.add(5)        # Add element
a.discard(5)    # Remove (no error if missing)
a.remove(1)     # Remove (KeyError if missing)
```

## Common Pitfalls
1. **Using `d[key]` without checking existence** -- This raises `KeyError` when the key is missing. Use `d.get(key, default)` or `d.setdefault(key, default)` for safe access.
2. **Using a mutable default with `fromkeys`** -- `dict.fromkeys(keys, [])` makes every key share the same list object. Use a dict comprehension instead: `{k: [] for k in keys}`.
3. **Confusing `discard` and `remove` on sets** -- `remove` raises `KeyError` if the element is absent; `discard` silently does nothing. Choose based on whether a missing element is an error or expected.

## Best Practices
1. **Use `defaultdict` or `setdefault` for grouping patterns** -- They eliminate the boilerplate of checking whether a key exists before appending to a list.
2. **Use the `|` merge operator for combining configs** -- It is clearer and creates a new dict, avoiding mutation of the originals.

## Summary
- Dictionaries provide O(1) key-value lookups and are central to almost every Python program.
- Use `.get()`, `.setdefault()`, and `defaultdict` for safe access patterns.
- The `|` operator (Python 3.9+) merges dictionaries cleanly.
- Sets offer fast membership tests and mathematical operations like union, intersection, and difference.
- Never use mutable objects as default values in `fromkeys`; use a dict comprehension instead.

## Code Examples

**Collections module helpers**

```python
# defaultdict for automatic default values
from collections import defaultdict

word_count = defaultdict(int)
for word in ["apple", "banana", "apple"]:
    word_count[word] += 1
# {'apple': 2, 'banana': 1}

# Counter for counting
from collections import Counter
counts = Counter(["a", "b", "a", "c", "a"])
print(counts.most_common(2))  # [('a', 3), ('b', 1)]
```


## Resources

- [Mapping Types — dict](https://docs.python.org/3.14/library/stdtypes.html#mapping-types-dict) — Official Python 3.14 reference for dictionary methods and set operations

---

> 📘 *This lesson is part of the [Python Fundamentals: Modern 3.14 Edition](https://stanza.dev/courses/python) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*