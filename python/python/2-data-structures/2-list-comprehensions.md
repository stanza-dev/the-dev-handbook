---
source_course: "python"
source_lesson: "python-list-comprehensions"
---

# List Comprehensions

List comprehensions provide a concise way to create lists based on existing iterables.

## Basic Syntax

```python
# Traditional approach
squares = []
for x in range(10):
    squares.append(x ** 2)

# List comprehension
squares = [x ** 2 for x in range(10)]
```

## With Conditions

```python
# Filter with if
evens = [x for x in range(20) if x % 2 == 0]

# If-else (ternary)
labels = ["even" if x % 2 == 0 else "odd" for x in range(5)]
# ['even', 'odd', 'even', 'odd', 'even']
```

## Nested Comprehensions

```python
# Flatten a 2D list
matrix = [[1, 2], [3, 4], [5, 6]]
flat = [num for row in matrix for num in row]
# [1, 2, 3, 4, 5, 6]

# Create a matrix
matrix = [[i * j for j in range(5)] for i in range(5)]
```

## Dictionary and Set Comprehensions

```python
# Dict comprehension
square_dict = {x: x**2 for x in range(5)}
# {0: 0, 1: 1, 2: 4, 3: 9, 4: 16}

# Set comprehension
unique_lengths = {len(word) for word in ["hello", "world", "hi"]}
# {2, 5}
```

## Generator Expressions

Use parentheses instead of brackets for memory-efficient iteration:

```python
# Generator expression (lazy evaluation)
sum_of_squares = sum(x**2 for x in range(1000000))

# List comprehension would create entire list in memory
# sum([x**2 for x in range(1000000)])  # Uses more memory
```

## When to Use What

- **List comprehension**: When you need the full list
- **Generator expression**: When iterating once (sum, any, all)
- **Traditional loop**: Complex logic or side effects

## Code Examples

**Practical list comprehension patterns**

```python
# Practical examples
words = ["Hello", "World", "Python"]

# Transform
lower = [w.lower() for w in words]

# Filter and transform
long_upper = [w.upper() for w in words if len(w) > 4]

# Multiple conditions
result = [x for x in range(100) if x % 2 == 0 if x % 3 == 0]
# Divisible by both 2 and 3

# Walrus operator in comprehensions (3.8+)
results = [y for x in range(10) if (y := x ** 2) > 50]
```


---

> 📘 *This lesson is part of the [Python Fundamentals: Modern 3.15 Edition](https://stanza.dev/courses/python) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*