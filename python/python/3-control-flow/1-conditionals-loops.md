---
source_course: "python"
source_lesson: "python-conditionals-loops"
---

# Conditionals and Loops

## Introduction
Conditionals and loops are the building blocks of any program's logic. Python's versions are clean and readable by design, and they include a few unique features -- like the loop `else` clause -- that you will not find in most other languages.

## Key Concepts
- **`if` / `elif` / `else`**: Branch execution based on boolean conditions.
- **Ternary expression**: `value_if_true if condition else value_if_false` -- a one-line conditional.
- **`for` loop**: Iterates directly over any iterable (list, range, dict, file, etc.).
- **`while` loop**: Repeats as long as a condition is truthy.
- **Loop `else` clause**: Runs when a loop finishes without hitting `break`.

## Real World Context
Every request handler, data pipeline, and CLI tool relies on conditionals and loops. The loop `else` clause is a Python-specific pattern that simplifies search-and-not-found logic without extra boolean flags. Mastering `break`, `continue`, and `else` lets you write control flow that is both concise and immediately clear to other developers.

## Deep Dive

### Conditional Statements

```python
if condition:
    # runs if condition is truthy
elif other_condition:
    # runs if first was false and this is true
else:
    # runs if all above were false
```

#### Ternary Expression

```python
result = "yes" if condition else "no"

# Chained ternary (use sparingly)
grade = "A" if score >= 90 else "B" if score >= 80 else "C"
```

### For Loops

Iterate over any iterable:

```python
# Over a list
for item in [1, 2, 3]:
    print(item)

# With index using enumerate
for i, item in enumerate(["a", "b", "c"]):
    print(f"{i}: {item}")

# Over a range
for i in range(5):  # 0, 1, 2, 3, 4
    print(i)

# Over dictionary items
for key, value in d.items():
    print(f"{key}: {value}")
```

### While Loops

```python
while condition:
    # runs while condition is truthy
    if should_exit:
        break
    if should_skip:
        continue
```

### Loop Else Clause

The `else` block runs if the loop completes without `break`:

```python
for item in items:
    if item == target:
        print("Found!")
        break
else:
    print("Not found")  # Only if loop didn't break
```

## Common Pitfalls
1. **Chaining too many ternary expressions** -- `a if x else b if y else c` is hard to read. Use a regular `if/elif/else` block when you have more than two branches.
2. **Modifying a collection during iteration** -- Deleting or inserting items while looping over a list causes skipped elements or `RuntimeError` with dicts. Iterate over a copy or build a new collection.
3. **Confusing loop `else` with conditional `else`** -- The loop `else` runs when the loop exits normally (no `break`), not when the loop body is "false." Think of it as "no break."

## Best Practices
1. **Use `for` loops over `while` loops when the iterable is known** -- `for item in items` is clearer and less error-prone than manually managing an index with `while`.
2. **Use `enumerate` instead of `range(len(...))`** -- It pairs each element with its index, eliminating off-by-one errors.

## Summary
- Python's `if/elif/else` and ternary expressions handle all branching needs.
- `for` loops iterate directly over iterables; `while` loops repeat until a condition is false.
- The loop `else` clause runs only when the loop completes without `break` -- useful for search patterns.
- Avoid modifying collections during iteration and keep ternary chains short.
- Prefer `for` with `enumerate` over `while` with manual index tracking.

## Code Examples

**Loop patterns**

```python
# Common patterns
numbers = [1, 2, 3, 4, 5]

# Find first match
for n in numbers:
    if n > 3:
        result = n
        break
else:
    result = None

# Enumerate with start index
for rank, player in enumerate(players, start=1):
    print(f"#{rank}: {player}")
```


## Resources

- [Control Flow](https://docs.python.org/3.14/tutorial/controlflow.html) — Official Python 3.14 tutorial on if statements, for loops, and control flow tools

---

> 📘 *This lesson is part of the [Python Fundamentals: Modern 3.14 Edition](https://stanza.dev/courses/python) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*