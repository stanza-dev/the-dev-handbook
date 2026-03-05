---
source_course: "python-performance"
source_lesson: "python-performance-slots-memory-optimization"
---

# Using __slots__ for Memory

## Introduction

Every Python object stores its attributes in a `__dict__` dictionary by default, adding significant per-instance overhead. The `__slots__` mechanism replaces this dictionary with a fixed-size struct, drastically reducing memory for classes with many instances.

## Key Concepts

- **`__slots__`**: A class-level declaration that tells Python to allocate space for a fixed set of attributes without creating a per-instance `__dict__`.
- **`__dict__` overhead**: The default attribute dictionary costs roughly 100+ bytes per instance (the dict object itself plus its internal hash table).
- **Descriptor-based access**: Slotted attributes are implemented as data descriptors on the class, providing direct memory offsets instead of hash lookups.
- **`dataclass(slots=True)`**: Python 3.10+ shorthand that automatically generates `__slots__` for dataclass fields.

## Real World Context

When loading millions of records into memory (e.g., a geospatial dataset of city coordinates, an in-memory graph of user nodes, or a particle simulation), reducing per-object overhead from ~160 bytes to ~56 bytes can mean the difference between 1.5 GB and 500 MB of RAM.

## Deep Dive

The standard Python object stores attributes in a per-instance dictionary. We can measure the cost directly:

```python
import sys

class WithDict:
    def __init__(self, x, y, z):
        self.x = x
        self.y = y
        self.z = z

obj = WithDict(1, 2, 3)
print(sys.getsizeof(obj))             # ~48 bytes (object shell)
print(sys.getsizeof(obj.__dict__))    # ~104 bytes (attribute dict)
# Total: ~152 bytes per instance
```

The object shell is small, but each instance carries its own dictionary. Now compare with `__slots__`:

```python
class WithSlots:
    __slots__ = ('x', 'y', 'z')
    def __init__(self, x, y, z):
        self.x = x
        self.y = y
        self.z = z

obj = WithSlots(1, 2, 3)
print(sys.getsizeof(obj))  # ~64 bytes total, no __dict__
```

Because there is no `__dict__`, you cannot add attributes that are not declared in `__slots__`:

```python
p = WithSlots(1, 2, 3)
p.w = 4  # Raises AttributeError: 'WithSlots' object has no attribute 'w'
```

This restriction is the tradeoff: you lose dynamic attribute assignment in exchange for lower memory and slightly faster access.

### Inheritance Rules

When subclassing a slotted class, declare only the new attributes in the child:

```python
class Base:
    __slots__ = ('a', 'b')

class Derived(Base):
    __slots__ = ('c',)  # Only new attributes; 'a' and 'b' are inherited

# Repeating parent slots creates redundant descriptors and wastes memory:
class Bad(Base):
    __slots__ = ('a', 'b', 'c')  # 'a' and 'b' are duplicated!
```

If any class in the hierarchy does not define `__slots__`, instances will still get a `__dict__` from that class, negating the memory savings.

### Modern Approach: dataclass(slots=True)

Python 3.10+ makes slotted classes trivial:

```python
from dataclasses import dataclass

@dataclass(slots=True)
class Point:
    x: float
    y: float
    z: float

p = Point(1.0, 2.0, 3.0)
print(sys.getsizeof(p))  # ~64 bytes, same savings
```

## Common Pitfalls

- **Forgetting that `__slots__` prevents `__dict__`**: Code that relies on `vars(obj)` or `obj.__dict__` will break. If you need both slots and a dict, include `'__dict__'` in `__slots__`, but this defeats much of the purpose.
- **Duplicating parent slots in child classes**: Repeating a parent's slot name in a subclass creates a redundant descriptor. Always declare only new attributes.
- **Assuming slots speed up attribute access significantly**: The primary benefit is memory reduction. Access speed improvement is marginal (5-10%) and should not be the sole motivation.

## Best Practices

- Use `__slots__` when you will create thousands or millions of instances with a fixed set of attributes.
- Prefer `@dataclass(slots=True)` for new code in Python 3.10+ to avoid manual `__slots__` declarations.
- Always measure with `sys.getsizeof()` or `tracemalloc` to confirm actual savings in your specific use case.

## Summary

- `__slots__` replaces the per-instance `__dict__` with fixed-offset attribute storage, saving ~100 bytes per object.
- Slotted classes cannot accept dynamic attributes unless `'__dict__'` is explicitly included in `__slots__`.
- In inheritance hierarchies, only declare new attributes in child `__slots__` to avoid redundant descriptors.
- `@dataclass(slots=True)` is the modern, clean way to create slotted classes in Python 3.10+.

## Code Examples

**Comparing per-instance memory between a regular class and a slotted class, then projecting savings at scale**

```python
import sys

class WithDict:
    def __init__(self, x, y, z):
        self.x, self.y, self.z = x, y, z

class WithSlots:
    __slots__ = ('x', 'y', 'z')
    def __init__(self, x, y, z):
        self.x, self.y, self.z = x, y, z

dict_obj = WithDict(1, 2, 3)
slot_obj = WithSlots(1, 2, 3)

dict_total = sys.getsizeof(dict_obj) + sys.getsizeof(dict_obj.__dict__)
slot_total = sys.getsizeof(slot_obj)

print(f"WithDict total:  {dict_total} bytes")
print(f"WithSlots total: {slot_total} bytes")
print(f"Savings per obj: {dict_total - slot_total} bytes")
print(f"For 1M objects:  {(dict_total - slot_total) * 1_000_000 / 1024 / 1024:.1f} MB saved")
```

**Using dataclass(slots=True) in Python 3.10+ for automatic __slots__ generation**

```python
from dataclasses import dataclass
import sys

@dataclass(slots=True)
class Point:
    x: float
    y: float
    z: float

p = Point(1.0, 2.0, 3.0)
print(f"Slotted dataclass size: {sys.getsizeof(p)} bytes")
print(f"Has __dict__: {hasattr(p, '__dict__')}")
print(f"Has __slots__: {hasattr(p, '__slots__')}")
print(f"Slots: {p.__slots__}")
```


## Resources

- [Data Model: __slots__](https://docs.python.org/3.14/reference/datamodel.html#slots) — Official documentation for __slots__ and its interaction with the data model
- [dataclasses — Data Classes](https://docs.python.org/3.14/library/dataclasses.html) — Official dataclasses module documentation including the slots parameter

---

> 📘 *This lesson is part of the [Python Performance: JIT & Internals](https://stanza.dev/courses/python-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*