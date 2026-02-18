---
source_course: "python-performance"
source_lesson: "python-performance-slots-memory-optimization"
---

# Using __slots__

By default, Python objects store attributes in a `__dict__`. `__slots__` eliminates this overhead.

## Memory Comparison

```python
import sys

class WithDict:
    def __init__(self, x, y):
        self.x = x
        self.y = y

class WithSlots:
    __slots__ = ('x', 'y')
    def __init__(self, x, y):
        self.x = x
        self.y = y

obj1 = WithDict(1, 2)
obj2 = WithSlots(1, 2)

print(sys.getsizeof(obj1))  # ~56 bytes + __dict__
print(sys.getsizeof(obj2))  # ~48 bytes
```

## Limitations

```python
class Point:
    __slots__ = ('x', 'y')

p = Point()
p.x = 10
p.y = 20
p.z = 30  # AttributeError!

# No __dict__
print(p.__dict__)  # AttributeError!
```

## Slots with Inheritance

```python
class Base:
    __slots__ = ('a',)

class Derived(Base):
    __slots__ = ('b',)  # Adds to parent slots

# Don't repeat parent slots!
class Bad(Base):
    __slots__ = ('a', 'b')  # 'a' creates redundant slot!
```

## Modern: slots=True with dataclass

```python
from dataclasses import dataclass

@dataclass(slots=True)  # Python 3.10+
class Point:
    x: float
    y: float
```

## Code Examples

**Memory savings with slots**

```python
import sys

# Compare memory for 1 million objects
class WithDict:
    def __init__(self, x, y, z):
        self.x, self.y, self.z = x, y, z

class WithSlots:
    __slots__ = ('x', 'y', 'z')
    def __init__(self, x, y, z):
        self.x, self.y, self.z = x, y, z

# Memory usage per object
dict_obj = WithDict(1, 2, 3)
slot_obj = WithSlots(1, 2, 3)

dict_size = sys.getsizeof(dict_obj) + sys.getsizeof(dict_obj.__dict__)
slot_size = sys.getsizeof(slot_obj)

print(f"WithDict: {dict_size} bytes")
print(f"WithSlots: {slot_size} bytes")
print(f"Savings: {dict_size - slot_size} bytes per object")
```


---

> 📘 *This lesson is part of the [Python Performance: JIT & Internals](https://stanza.dev/courses/python-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*