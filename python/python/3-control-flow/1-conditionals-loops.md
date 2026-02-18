---
source_course: "python"
source_lesson: "python-conditionals-loops"
---

# Control Flow Basics

## Conditional Statements

```python
if condition:
    # runs if condition is truthy
elif other_condition:
    # runs if first was false and this is true
else:
    # runs if all above were false
```

### Ternary Expression

```python
result = "yes" if condition else "no"

# Chained ternary (use sparingly)
grade = "A" if score >= 90 else "B" if score >= 80 else "C"
```

## For Loops

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

## While Loops

```python
while condition:
    # runs while condition is truthy
    if should_exit:
        break
    if should_skip:
        continue
```

## Loop Else Clause

The `else` block runs if the loop completes without `break`:

```python
for item in items:
    if item == target:
        print("Found!")
        break
else:
    print("Not found")  # Only if loop didn't break
```

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


---

> 📘 *This lesson is part of the [Python Fundamentals: Modern 3.15 Edition](https://stanza.dev/courses/python) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*