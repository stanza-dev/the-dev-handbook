---
source_course: "python"
source_lesson: "python-builtin-functions"
---

# Essential Built-in Functions

Python provides powerful built-in functions for working with sequences.

## Iteration Helpers

### `enumerate()`
```python
fruits = ["apple", "banana", "cherry"]
for i, fruit in enumerate(fruits):
    print(f"{i}: {fruit}")

# Start from different index
for i, fruit in enumerate(fruits, start=1):
    print(f"{i}: {fruit}")
```

### `zip()`
```python
names = ["Alice", "Bob"]
ages = [30, 25]

for name, age in zip(names, ages):
    print(f"{name} is {age}")

# Unzip
pairs = [("a", 1), ("b", 2)]
letters, numbers = zip(*pairs)
```

### `reversed()` and `sorted()`
```python
lst = [3, 1, 4, 1, 5]

list(reversed(lst))  # [5, 1, 4, 1, 3]
sorted(lst)          # [1, 1, 3, 4, 5] (new list)
sorted(lst, reverse=True)  # [5, 4, 3, 1, 1]

# Custom sorting
words = ["banana", "pie", "apple"]
sorted(words, key=len)  # ['pie', 'apple', 'banana']
```

## Aggregation Functions

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

## Boolean Functions

```python
nums = [1, 2, 0, 4]

all(nums)       # False (0 is falsy)
any(nums)       # True (some are truthy)

all(x > 0 for x in [1, 2, 3])  # True
any(x < 0 for x in [1, 2, -3])  # True
```

## Transformation Functions

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


---

> 📘 *This lesson is part of the [Python Fundamentals: Modern 3.15 Edition](https://stanza.dev/courses/python) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*