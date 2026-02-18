---
source_course: "python-metaprogramming"
source_lesson: "python-metaprogramming-descriptor-protocol"
---

# Descriptors

Descriptors allow objects to customize attribute lookup, storage, and deletion. They power `@property`, `@classmethod`, and `super()`.

## The Protocol
To define a descriptor, implement `__get__`, `__set__`, and/or `__delete__`.

```python
class PositiveNumber:
    def __set_name__(self, owner, name):
        self.name = name

    def __get__(self, obj, objtype=None):
        return obj.__dict__.get(self.name)

    def __set__(self, obj, value):
        if value < 0:
            raise ValueError("Must be positive")
        obj.__dict__[self.name] = value

class Wallet:
    balance = PositiveNumber()
```

## Code Examples

**Lazy Property Descriptor**

```python
class Lazy:
    def __init__(self, func):
        self.func = func
        self.name = func.__name__

    def __get__(self, obj, cls):
        if obj is None:
            return self
        value = self.func(obj)
        setattr(obj, self.name, value)
        return value

class Data:
    @Lazy
    def heavy_computation(self):
        return sum(range(10**6))
```


---

> 📘 *This lesson is part of the [Python Metaprogramming & Introspection](https://stanza.dev/courses/python-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*