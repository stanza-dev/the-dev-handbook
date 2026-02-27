---
source_course: "python"
source_lesson: "python-list-comprehensions"
---

# List Comprehensions & Generator Expressions

## Introduction
List comprehensions are one of Python's most distinctive features -- a concise, readable way to create lists from existing iterables. This lesson also covers generator expressions, which provide the same power with lazy evaluation for memory efficiency.

## Key Concepts
- **List comprehension**: `[expr for item in iterable if condition]` -- builds a list in one expression.
- **Generator expression**: `(expr for item in iterable if condition)` -- produces values lazily, one at a time.
- **Dict comprehension**: `{key: value for item in iterable}` -- builds a dictionary.
- **Set comprehension**: `{expr for item in iterable}` -- builds a set of unique values.

## Real World Context
Comprehensions appear in nearly every production Python codebase. Data pipelines use them to transform rows, web frameworks use them to filter query results, and CLI tools use them to process file listings. Choosing between a list comprehension and a generator expression can mean the difference between a script that runs in megabytes of memory and one that crashes on large datasets.

## Deep Dive

### Basic Syntax

```python
# Traditional approach
squares = []
for x in range(10):
    squares.append(x ** 2)

# List comprehension
squares = [x ** 2 for x in range(10)]
```

### With Conditions

```python
# Filter with if
evens = [x for x in range(20) if x % 2 == 0]

# If-else (ternary)
labels = ["even" if x % 2 == 0 else "odd" for x in range(5)]
# ['even', 'odd', 'even', 'odd', 'even']
```

### Nested Comprehensions

```python
# Flatten a 2D list
matrix = [[1, 2], [3, 4], [5, 6]]
flat = [num for row in matrix for num in row]
# [1, 2, 3, 4, 5, 6]

# Create a matrix
matrix = [[i * j for j in range(5)] for i in range(5)]
```

### Dictionary and Set Comprehensions

```python
# Dict comprehension
square_dict = {x: x**2 for x in range(5)}
# {0: 0, 1: 1, 2: 4, 3: 9, 4: 16}

# Set comprehension
unique_lengths = {len(word) for word in ["hello", "world", "hi"]}
# {2, 5}
```

### Generator Expressions

Use parentheses instead of brackets for memory-efficient iteration:

```python
# Generator expression (lazy evaluation)
sum_of_squares = sum(x**2 for x in range(1000000))

# List comprehension would create entire list in memory
# sum([x**2 for x in range(1000000)])  # Uses more memory
```

### When to Use What

- **List comprehension**: When you need the full list
- **Generator expression**: When iterating once (sum, any, all)
- **Traditional loop**: Complex logic or side effects

## Common Pitfalls
1. **Putting the if-else in the wrong position** -- Filtering (`if` only) goes after the `for`: `[x for x in items if x > 0]`. A ternary (`if-else`) goes before the `for`: `[x if x > 0 else 0 for x in items]`. Mixing them up is a syntax error.
2. **Writing overly complex comprehensions** -- A comprehension with three nested loops and two conditions is harder to read than a plain loop. If it does not fit on one line comfortably, use a regular loop.
3. **Using a list comprehension when you only need to iterate** -- `sum([x**2 for x in data])` builds an entire list just to throw it away. Use a generator: `sum(x**2 for x in data)`.

## Best Practices
1. **Keep comprehensions simple** -- One loop and one filter is the sweet spot. Beyond that, break it into a loop or a helper function.
2. **Use generator expressions with aggregation functions** -- `sum()`, `any()`, `all()`, and `max()` all accept generators, saving memory.

## Summary
- List comprehensions provide a concise, Pythonic way to create lists from iterables.
- Generator expressions use lazy evaluation and are ideal when you iterate only once.
- Dict and set comprehensions follow the same pattern with `{}` syntax.
- Keep comprehensions readable; if they grow complex, switch to a traditional loop.
- Prefer generator expressions when feeding results directly into `sum()`, `any()`, or `all()`.

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


## Resources

- [List Comprehensions](https://docs.python.org/3.14/tutorial/datastructures.html#list-comprehensions) — Official Python 3.14 tutorial section on list comprehensions and generator expressions

---

> 📘 *This lesson is part of the [Python Fundamentals: Modern 3.14 Edition](https://stanza.dev/courses/python) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*