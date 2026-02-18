---
source_course: "python"
source_lesson: "python-lists-and-tuples"
---

# Lists vs Tuples

Python provides two fundamental sequence types: mutable lists and immutable tuples.

## Lists (`list`)

Lists are mutable, ordered sequences. They can contain any type of object and can grow or shrink dynamically.

```python
# Creating lists
numbers = [1, 2, 3, 4, 5]
mixed = [1, "hello", 3.14, True]
empty = []
from_range = list(range(5))  # [0, 1, 2, 3, 4]
```

### List Operations

```python
lst = [1, 2, 3]

# Adding elements
lst.append(4)         # [1, 2, 3, 4]
lst.insert(0, 0)      # [0, 1, 2, 3, 4]
lst.extend([5, 6])    # [0, 1, 2, 3, 4, 5, 6]

# Removing elements
lst.pop()             # Returns 6, lst = [0, 1, 2, 3, 4, 5]
lst.remove(0)         # [1, 2, 3, 4, 5]
del lst[0]            # [2, 3, 4, 5]

# Other operations
lst.reverse()         # Reverses in place
lst.sort()            # Sorts in place
lst.index(3)          # Find index of value
lst.count(3)          # Count occurrences
```

## Tuples (`tuple`)

Tuples are immutable sequences. Once created, they cannot be modified.

```python
coordinates = (10, 20)
singleton = (42,)      # Note the comma!
empty = ()
from_list = tuple([1, 2, 3])
```

### When to Use Tuples

- Function return values (multiple values)
- Dictionary keys (lists can't be keys)
- Data that shouldn't change
- Slightly faster than lists

## Unpacking

```python
# Basic unpacking
x, y = (10, 20)
a, b, c = [1, 2, 3]

# Extended unpacking with *
first, *middle, last = [1, 2, 3, 4, 5]
# first=1, middle=[2,3,4], last=5

# Ignoring values with _
x, _, z = (1, 2, 3)  # Ignore second value
```

## Code Examples

**Advanced tuple usage**

```python
# Tuple unpacking in loops
points = [(0, 0), (1, 1), (2, 4)]
for x, y in points:
    print(f"x={x}, y={y}")

# Swapping variables
a, b = 1, 2
a, b = b, a  # a=2, b=1

# Named tuples for clarity
from collections import namedtuple
Point = namedtuple('Point', ['x', 'y'])
p = Point(10, 20)
print(p.x, p.y)  # 10 20
```


---

> 📘 *This lesson is part of the [Python Fundamentals: Modern 3.15 Edition](https://stanza.dev/courses/python) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*