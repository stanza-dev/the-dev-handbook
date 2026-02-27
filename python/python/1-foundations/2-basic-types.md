---
source_course: "python"
source_lesson: "python-basic-types"
---

# Basic Data Types

## Introduction
Every value in Python has a type, and understanding those types is the first step toward writing correct programs. This lesson covers the numeric, boolean, and None types you will use in virtually every Python script.

## Key Concepts
- **Dynamic typing**: Python infers a variable's type at runtime -- you never write `int x = 42`.
- **int**: Unlimited-precision integers.
- **float**: 64-bit IEEE 754 double-precision numbers.
- **complex**: Built-in complex number support with a `j` suffix.
- **bool**: `True` or `False`, actually a subclass of `int`.
- **None**: Python's null value, representing the absence of data.

## Real World Context
Misunderstanding Python's type system causes subtle bugs in production. For example, comparing `True == 1` evaluates to `True` because `bool` is a subclass of `int`. Knowing how truthiness works is essential for writing correct conditionals and filters in data pipelines, web handlers, and CLI tools.

## Deep Dive

### Integers (`int`)
Unlimited precision integers -- Python handles arbitrarily large numbers automatically.

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

### Boolean Type (`bool`)

Booleans are `True` or `False` (note the capitalization). They are actually a subclass of `int`.

```python
is_valid = True
print(True + True)  # 2 (booleans are integers!)
```

### None Type

`None` represents the absence of a value. It is Python's null equivalent.

```python
result = None
if result is None:
    print("No result yet")
```

### Type Checking

Use `type()` to check an object's type, and `isinstance()` for type checking that respects inheritance.

```python
type(42)           # <class 'int'>
isinstance(42, int)  # True
isinstance(True, int)  # True (bool is subclass of int)
```

## Common Pitfalls
1. **Floating-point precision errors** -- `0.1 + 0.2` does not equal `0.3` due to IEEE 754 representation. Use `math.isclose()` or the `decimal` module when exact decimal arithmetic matters.
2. **Confusing `==` with `is` for None checks** -- Always use `is None` instead of `== None`. The `is` operator checks identity, which is the correct way to test for None.
3. **Assuming `bool` behaves differently from `int`** -- Because `True == 1` and `False == 0`, arithmetic on booleans can produce surprising results if you forget this relationship.

## Best Practices
1. **Use `isinstance()` over `type()` for type checks** -- `isinstance` respects inheritance and works with abstract base classes, making your code more flexible.
2. **Use `is` for singleton comparisons** -- Compare against `None`, `True`, and `False` with `is`, not `==`.

## Summary
- Python is dynamically typed; you never declare types explicitly.
- Integers have unlimited precision, floats follow IEEE 754, and complex numbers are built in.
- `bool` is a subclass of `int`; `True` and `False` behave as 1 and 0 in arithmetic.
- `None` represents the absence of a value and should be checked with `is None`.
- Use `isinstance()` for type checking and `math.isclose()` for float comparisons.

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


## Resources

- [Built-in Types](https://docs.python.org/3.14/library/stdtypes.html) — Official Python 3.14 reference for built-in types including numeric, boolean, and None types

---

> 📘 *This lesson is part of the [Python Fundamentals: Modern 3.14 Edition](https://stanza.dev/courses/python) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*