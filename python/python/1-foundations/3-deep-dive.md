---
source_course: "python"
source_lesson: "python-strings-deep-dive"
---

# Strings in Python

Strings are immutable sequences of Unicode characters. Python 3 handles Unicode natively.

## String Creation

```python
single = 'Hello'
double = "World"
multiline = '''This is
a multiline
string'''
raw = r"C:\Users\name"  # Raw string (no escape processing)
```

## String Methods

Strings have many built-in methods:

```python
text = "  Hello World  "

text.strip()       # "Hello World"
text.lower()       # "  hello world  "
text.upper()       # "  HELLO WORLD  "
text.split()       # ["Hello", "World"]
text.replace("World", "Python")  # "  Hello Python  "
```

## String Formatting

Python offers multiple ways to format strings:

### F-Strings (Recommended)
```python
name = "Alice"
age = 30
print(f"{name} is {age} years old")
print(f"{age:03d}")  # "030" (zero-padded)
print(f"{3.14159:.2f}")  # "3.14" (2 decimal places)
```

### Format Specifiers
```python
# Width and alignment
f"{'left':<10}"   # 'left      '
f"{'right':>10}"  # '     right'
f"{'center':^10}" # '  center  '

# Number formatting
f"{1000000:,}"    # '1,000,000'
f"{0.5:.0%}"      # '50%'
```

## String Indexing and Slicing

```python
s = "Python"
s[0]      # 'P'
s[-1]     # 'n'
s[0:3]    # 'Pyt'
s[::2]    # 'Pto' (every second character)
s[::-1]   # 'nohtyP' (reversed)
```

## Code Examples

**Modern f-string features**

```python
# F-string debugging (3.8+)
x = 10
y = 20
print(f"{x=}, {y=}")  # "x=10, y=20"

# F-string expressions
items = ["apple", "banana"]
print(f"Count: {len(items)}")  # "Count: 2"

# Nested quotes (3.12+)
data = {"key": "value"}
print(f"Result: {data["key"]}")
```


---

> 📘 *This lesson is part of the [Python Fundamentals: Modern 3.15 Edition](https://stanza.dev/courses/python) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*