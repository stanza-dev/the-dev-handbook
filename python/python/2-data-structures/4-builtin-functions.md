---
source_course: "python"
source_lesson: "python-builtin-functions"
---

# Built-in Functions for Data Structures

## Introduction
Python ships with a rich set of built-in functions specifically designed for working with sequences and iterables. Mastering `enumerate`, `zip`, `sorted`, `any`, and `all` will make your code shorter, faster, and more Pythonic.

## Key Concepts
- **`enumerate(iterable, start=0)`**: Pairs each element with its index.
- **`zip(*iterables)`**: Combines elements from multiple iterables position by position.
- **`sorted(iterable, key=None, reverse=False)`**: Returns a new sorted list without modifying the original.
- **`any()` / `all()`**: Short-circuit boolean checks across an iterable.
- **`map()` / `filter()`**: Functional-style transformation and filtering (comprehensions are often preferred).

## Real World Context
These built-ins are the bread and butter of data processing in Python. A web handler might use `any()` to check if a user has any required permission, `sorted()` with a `key` to rank search results, or `zip()` to pair column headers with row values from a CSV. Knowing them by heart eliminates the need for manual index tracking and verbose loops.

## Deep Dive

### Iteration Helpers

#### `enumerate()`
```python
fruits = ["apple", "banana", "cherry"]
for i, fruit in enumerate(fruits):
    print(f"{i}: {fruit}")

# Start from different index
for i, fruit in enumerate(fruits, start=1):
    print(f"{i}: {fruit}")
```

#### `zip()`
```python
names = ["Alice", "Bob"]
ages = [30, 25]

for name, age in zip(names, ages):
    print(f"{name} is {age}")

# Unzip
pairs = [("a", 1), ("b", 2)]
letters, numbers = zip(*pairs)
```

#### `reversed()` and `sorted()`
```python
lst = [3, 1, 4, 1, 5]

list(reversed(lst))  # [5, 1, 4, 1, 3]
sorted(lst)          # [1, 1, 3, 4, 5] (new list)
sorted(lst, reverse=True)  # [5, 4, 3, 1, 1]

# Custom sorting
words = ["banana", "pie", "apple"]
sorted(words, key=len)  # ['pie', 'apple', 'banana']
```

### Aggregation Functions

```python
nums = [1, 2, 3, 4, 5]

sum(nums)       # 15
min(nums)       # 1
max(nums)       # 5
len(nums)       # 5

# With key function
words = ["hi", "hello", "hey"]
max(words, key=len)  # "hello"
```

### Boolean Functions

```python
nums = [1, 2, 0, 4]

all(nums)       # False (0 is falsy)
any(nums)       # True (some are truthy)

all(x > 0 for x in [1, 2, 3])  # True
any(x < 0 for x in [1, 2, -3])  # True
```

### Transformation Functions

```python
# map() - apply function to all elements
nums = [1, 2, 3]
squared = list(map(lambda x: x**2, nums))  # [1, 4, 9]

# filter() - keep elements where function returns True
evens = list(filter(lambda x: x % 2 == 0, range(10)))
# [0, 2, 4, 6, 8]

# Note: List comprehensions are often more Pythonic
squared = [x**2 for x in nums]
evens = [x for x in range(10) if x % 2 == 0]
```

## Common Pitfalls
1. **Using `range(len(lst))` instead of `enumerate`** -- Writing `for i in range(len(lst))` is verbose and error-prone. Use `for i, item in enumerate(lst)` instead.
2. **Forgetting that `zip` stops at the shortest iterable** -- If your lists have different lengths, `zip` silently drops extra elements. Use `itertools.zip_longest` when you need to handle unequal lengths.
3. **Calling `sorted()` when you need in-place sorting** -- `sorted()` creates a new list. If you want to sort in place and save memory, use `lst.sort()`.

## Best Practices
1. **Prefer comprehensions over `map`/`filter` with lambdas** -- `[x**2 for x in nums]` is more readable than `list(map(lambda x: x**2, nums))`.
2. **Use generator expressions with `any`/`all`/`sum`** -- They short-circuit or stream without building intermediate lists.

## Summary
- `enumerate` and `zip` eliminate manual index management and parallel iteration boilerplate.
- `sorted` returns a new list; `list.sort()` sorts in place.
- `any()` and `all()` short-circuit for efficient boolean checks on iterables.
- Prefer list comprehensions over `map`/`filter` with lambdas for readability.
- Use `itertools.zip_longest` when combining iterables of different lengths.

## Code Examples

**Combining built-in functions**

```python
# Combining functions
data = [("Alice", 85), ("Bob", 92), ("Charlie", 78)]

# Sort by score descending
sorted_data = sorted(data, key=lambda x: x[1], reverse=True)

# Find top scorer
top = max(data, key=lambda x: x[1])
print(f"Top scorer: {top[0]} with {top[1]}")

# Check if all passed (score >= 60)
all_passed = all(score >= 60 for _, score in data)
```


## Resources

- [Built-in Functions](https://docs.python.org/3.14/library/functions.html) — Official Python 3.14 reference for all built-in functions like enumerate, zip, sorted, and more

---

> 📘 *This lesson is part of the [Python Fundamentals: Modern 3.14 Edition](https://stanza.dev/courses/python) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*