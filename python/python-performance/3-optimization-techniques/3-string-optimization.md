---
source_course: "python-performance"
source_lesson: "python-performance-string-optimization"
---

# String Performance

## String Concatenation

```python
# Bad: O(n²) - creates new string each time
result = ""
for word in words:
    result += word  # Slow!

# Good: O(n)
result = "".join(words)
```

## Building Strings

```python
# For many small additions, use list
parts = []
for item in items:
    parts.append(f"{item}: {item.value}")
result = "\n".join(parts)

# Or use io.StringIO
from io import StringIO

buffer = StringIO()
for item in items:
    buffer.write(f"{item}: {item.value}\n")
result = buffer.getvalue()
```

## Collection Optimization

```python
# Use tuple for fixed collections
point = (10, 20)  # Immutable, less memory

# Use set for membership testing
huge_list = [...]
if item in huge_list:  # O(n)
    pass

huge_set = set(huge_list)
if item in huge_set:  # O(1)
    pass

# Use dict.fromkeys for unique items in order
unique_ordered = list(dict.fromkeys(items))
```

## Generator Expressions

```python
# Memory-inefficient
sum([x**2 for x in range(1000000)])  # Creates list

# Memory-efficient
sum(x**2 for x in range(1000000))  # Generator
```

## Code Examples

**String concatenation benchmark**

```python
import timeit

words = ["word"] * 10000

# Benchmark string concatenation methods
def concat_plus():
    result = ""
    for w in words:
        result += w
    return result

def concat_join():
    return "".join(words)

print(f"Plus: {timeit.timeit(concat_plus, number=100):.4f}s")
print(f"Join: {timeit.timeit(concat_join, number=100):.4f}s")
```


---

> 📘 *This lesson is part of the [Python Performance: JIT & Internals](https://stanza.dev/courses/python-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*