---
source_course: "python"
source_lesson: "python-dictionaries-and-sets"
---

# Dictionaries

Dictionaries are key-value mappings with O(1) average lookup time.

## Creating Dictionaries

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

## Dictionary Operations

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

## Dictionary Views

```python
d = {"a": 1, "b": 2}

d.keys()    # dict_keys(['a', 'b'])
d.values()  # dict_values([1, 2])
d.items()   # dict_items([('a', 1), ('b', 2)])

# Iteration
for key, value in d.items():
    print(f"{key}: {value}")
```

## Dictionary Merging (3.9+)

```python
defaults = {"theme": "light", "lang": "en"}
user = {"theme": "dark"}

# Union operator (creates new dict)
config = defaults | user  # {'theme': 'dark', 'lang': 'en'}

# In-place update
defaults |= user
```

# Sets

Sets are unordered collections of unique elements.

## Set Operations

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


---

> 📘 *This lesson is part of the [Python Fundamentals: Modern 3.15 Edition](https://stanza.dev/courses/python) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*