---
source_course: "python"
source_lesson: "python-lists-and-tuples"
---

# Lists and Tuples

## Introduction
Lists and tuples are the two fundamental sequence types in Python. Knowing when to reach for a mutable list versus an immutable tuple is a skill you will use in every Python project, from scripts to large-scale applications.

## Key Concepts
- **list**: A mutable, ordered sequence that can grow or shrink dynamically.
- **tuple**: An immutable, ordered sequence whose contents cannot change after creation.
- **Unpacking**: Assigning elements of a sequence to individual variables in a single statement.
- **Extended unpacking (`*`)**: Capturing multiple elements into a sub-list during unpacking.

## Real World Context
Functions that return multiple values use tuples by default (`return x, y`). Dictionary keys must be hashable, so you can use tuples as compound keys but never lists. In data-heavy code, choosing the right sequence type affects both correctness and performance -- tuples are slightly faster and use less memory.

## Deep Dive

### Lists (`list`)

Lists are mutable, ordered sequences. They can contain any type of object and can grow or shrink dynamically.

```python
# Creating lists
numbers = [1, 2, 3, 4, 5]
mixed = [1, "hello", 3.14, True]
empty = []
from_range = list(range(5))  # [0, 1, 2, 3, 4]
```

#### List Operations

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

### Tuples (`tuple`)

Tuples are immutable sequences. Once created, they cannot be modified.

```python
coordinates = (10, 20)
singleton = (42,)      # Note the comma!
empty = ()
from_list = tuple([1, 2, 3])
```

#### When to Use Tuples

- Function return values (multiple values)
- Dictionary keys (lists cannot be keys)
- Data that should not change
- Slightly faster than lists

### Unpacking

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

## Common Pitfalls
1. **Forgetting the comma in single-element tuples** -- `(42)` is just an integer in parentheses. You need `(42,)` to create a tuple.
2. **Confusing `append` with `extend`** -- `lst.append([1, 2])` adds the list as a single element. `lst.extend([1, 2])` adds each element individually.
3. **Modifying a list while iterating over it** -- Removing or inserting items during iteration can skip elements or raise errors. Iterate over a copy or build a new list instead.

## Best Practices
1. **Default to tuples for fixed collections** -- If the data will not change, use a tuple. It communicates intent and enables use as dict keys or set members.
2. **Use unpacking to write clearer code** -- `x, y = point` is more readable than `x = point[0]; y = point[1]`.

## Summary
- Lists are mutable sequences ideal for collections that change over time.
- Tuples are immutable sequences perfect for fixed data, function return values, and dictionary keys.
- Unpacking with `*` lets you capture variable numbers of elements elegantly.
- Always use a trailing comma for single-element tuples: `(42,)`.
- Prefer tuples when the data will not be modified.

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


## Resources

- [Data Structures Tutorial](https://docs.python.org/3.14/tutorial/datastructures.html) — Official Python 3.14 tutorial covering lists, tuples, and sequence operations

---

> 📘 *This lesson is part of the [Python Fundamentals: Modern 3.14 Edition](https://stanza.dev/courses/python) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*