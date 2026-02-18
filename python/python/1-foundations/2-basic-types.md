---
source_course: "python"
source_lesson: "python-basic-types"
---

# Python's Basic Data Types

Python is dynamically typed, meaning you don't declare types explicitly. However, understanding types is crucial for writing correct code.

## Numeric Types

### Integers (`int`)
Unlimited precision integers - Python handles arbitrarily large numbers automatically.

```python
x = 42
big = 10 ** 100  # No overflow!
hex_val = 0xFF   # 255 in hexadecimal
binary = 0b1010  # 10 in binary
```

### Floating Point (`float`)
64-bit IEEE 754 double precision numbers.

```python
pi = 3.14159
scientific = 1.5e-10  # 0.00000000015
```

### Complex Numbers (`complex`)
Built-in support for complex arithmetic.

```python
z = 3 + 4j
print(z.real, z.imag)  # 3.0 4.0
```

## Boolean Type (`bool`)

Booleans are `True` or `False` (note the capitalization). They're actually a subclass of `int`.

```python
is_valid = True
print(True + True)  # 2 (booleans are integers!)
```

## None Type

`None` represents the absence of a value. It's Python's null equivalent.

```python
result = None
if result is None:
    print("No result yet")
```

## Type Checking

Use `type()` to check an object's type, and `isinstance()` for type checking that respects inheritance.

```python
type(42)           # <class 'int'>
isinstance(42, int)  # True
isinstance(True, int)  # True (bool is subclass of int)
```

## Code Examples

**Numeric operations and type conversion**

```python
# Numeric operations
x = 10
y = 3

print(x / y)   # 3.333... (true division)
print(x // y)  # 3 (floor division)
print(x % y)   # 1 (modulo)
print(x ** y)  # 1000 (power)

# Type conversion
print(int(3.7))    # 3
print(float(42))   # 42.0
print(bool(0))     # False
print(bool(1))     # True
```


---

> 📘 *This lesson is part of the [Python Fundamentals: Modern 3.15 Edition](https://stanza.dev/courses/python) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*