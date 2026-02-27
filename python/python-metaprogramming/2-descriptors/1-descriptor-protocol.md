---
source_course: "python-metaprogramming"
source_lesson: "python-metaprogramming-descriptor-protocol"
---

# The Descriptor Protocol

## Introduction

Descriptors are one of Python's most powerful and least understood features. They are the mechanism behind `@property`, `@classmethod`, `@staticmethod`, `super()`, and even how ordinary method calls work. Any time you access an attribute on a Python object, the descriptor protocol may be silently at work.

A descriptor is simply an object that defines one or more of the special methods `__get__`, `__set__`, or `__delete__`. When such an object is stored as a **class attribute**, Python intercepts attribute access and delegates to the descriptor's methods instead of performing a normal dictionary lookup.

## Key Concepts

- **`__get__(self, obj, objtype=None)`** -- Called when the attribute is read. `obj` is the instance (or `None` if accessed from the class), and `objtype` is always the owner class.
- **`__set__(self, obj, value)`** -- Called when the attribute is assigned a value.
- **`__delete__(self, obj)`** -- Called when the attribute is deleted with `del`.
- **`__set_name__(self, owner, name)`** -- Called at class creation time (Python 3.6+) to tell the descriptor what attribute name it was assigned to.

A descriptor must live on the **class**, not on the instance, for the protocol to activate.

## Real World Context

Descriptors are everywhere in production Python:
- Django's model fields (`CharField`, `IntegerField`) are descriptors.
- SQLAlchemy's `Column` objects use the descriptor protocol for attribute-mapped ORM.
- Python's own `property`, `classmethod`, and `staticmethod` are implemented as descriptors.
- `functools.cached_property` is a non-data descriptor.

## Deep Dive

Let's build a descriptor from scratch that validates positive numbers:

```python
class PositiveNumber:
    """A descriptor that only allows positive numeric values."""

    def __set_name__(self, owner, name):
        # Called automatically at class creation time
        self.name = name
        self.private_name = f'_descriptor_{name}'

    def __get__(self, obj, objtype=None):
        if obj is None:
            # Accessed from the class itself, e.g. Wallet.balance
            return self
        return getattr(obj, self.private_name, None)

    def __set__(self, obj, value):
        if not isinstance(value, (int, float)):
            raise TypeError(f"{self.name} must be a number")
        if value < 0:
            raise ValueError(f"{self.name} must be positive")
        setattr(obj, self.private_name, value)

    def __delete__(self, obj):
        if hasattr(obj, self.private_name):
            delattr(obj, self.private_name)
        else:
            raise AttributeError(f"{self.name} not set")


class Wallet:
    balance = PositiveNumber()

    def __init__(self, balance):
        self.balance = balance  # Calls PositiveNumber.__set__

w = Wallet(100)
print(w.balance)   # 100 -- calls PositiveNumber.__get__
w.balance = 200    # OK -- calls PositiveNumber.__set__
# w.balance = -50  # ValueError: balance must be positive
```

Notice that `__set_name__` removes the need to pass the attribute name manually. Before Python 3.6, you had to write `balance = PositiveNumber('balance')`, which was error-prone and repetitive.

## How Python Triggers Descriptors

When you write `obj.attr`, Python's attribute lookup roughly follows this order:

1. Call `type(obj).__mro__` to find the class and its bases.
2. Look for `attr` in the class (and its MRO). If found and it is a **data descriptor** (has `__get__` and `__set__` or `__delete__`), call its `__get__` and return the result.
3. Look for `attr` in `obj.__dict__`. If found, return it.
4. If the class attribute is a **non-data descriptor** (has only `__get__`), call its `__get__` and return.
5. If none of the above, raise `AttributeError`.

## Common Pitfalls

- **Storing state on the descriptor instead of the instance.** If you write `self.value = value` inside `__set__`, all instances share the same value. Always store per-instance data on `obj` (the instance), not on `self` (the descriptor).
- **Forgetting that descriptors must be class attributes.** Putting a descriptor in `__init__` as an instance attribute does not trigger the protocol.
- **Not handling `obj is None` in `__get__`.** When the descriptor is accessed from the class (e.g., `MyClass.attr`), `obj` is `None`. Return `self` or a helpful message in that case.

## Best Practices

- Always implement `__set_name__` to capture the attribute name automatically.
- Store per-instance data using a mangled or prefixed private name on the instance, not on the descriptor itself.
- Return `self` from `__get__` when `obj is None` so the descriptor is inspectable from the class.
- Combine descriptors with `__init_subclass__` or metaclasses for framework-level validation.

## Summary

The descriptor protocol (`__get__`, `__set__`, `__delete__`, `__set_name__`) lets you intercept attribute access on instances. Descriptors must be class-level attributes. They are the foundation of `property`, `classmethod`, `staticmethod`, and many Python frameworks. Always store instance data on the instance (not the descriptor) and implement `__set_name__` for clean, automatic name capture.

## Code Examples

**Lazy property descriptor that computes once, then caches the result on the instance.**

```python
class Lazy:
    """Non-data descriptor for lazy evaluation.
    Computes the value on first access, then caches it
    directly in the instance __dict__ so subsequent
    accesses bypass the descriptor entirely."""

    def __init__(self, func):
        self.func = func
        self.name = func.__name__

    def __get__(self, obj, cls):
        if obj is None:
            return self
        # Compute and cache in instance dict
        value = self.func(obj)
        setattr(obj, self.name, value)
        return value


class Data:
    @Lazy
    def heavy_computation(self):
        print("Computing...")
        return sum(range(10**6))

d = Data()
print(d.heavy_computation)  # Computing... -> 499999500000
print(d.heavy_computation)  # 499999500000 (no recomputation)
```

**Type-checking descriptor that validates types on assignment.**

```python
class TypeChecked:
    """Descriptor enforcing type at assignment time."""

    def __init__(self, expected_type):
        self.expected_type = expected_type

    def __set_name__(self, owner, name):
        self.name = name

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return obj.__dict__.get(self.name)

    def __set__(self, obj, value):
        if not isinstance(value, self.expected_type):
            raise TypeError(
                f"{self.name} must be {self.expected_type.__name__}, "
                f"got {type(value).__name__}"
            )
        obj.__dict__[self.name] = value


class Person:
    name = TypeChecked(str)
    age = TypeChecked(int)

    def __init__(self, name, age):
        self.name = name
        self.age = age

p = Person("Alice", 30)   # OK
# Person("Alice", "old")  # TypeError: age must be int, got str
```


## Resources

- [Descriptor HowTo Guide](https://docs.python.org/3.14/howto/descriptor.html) — undefined
- [Data Model -- Implementing Descriptors](https://docs.python.org/3.14/reference/datamodel.html#descriptors) — undefined

---

> 📘 *This lesson is part of the [Python Metaprogramming & Introspection](https://stanza.dev/courses/python-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*