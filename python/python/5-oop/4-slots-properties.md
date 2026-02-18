---
source_course: "python"
source_lesson: "python-slots-properties"
---

# __slots__

By default, Python stores instance attributes in a `__dict__`. Using `__slots__` restricts attributes to a fixed set, saving memory.

```python
class Point:
    __slots__ = ('x', 'y')
    
    def __init__(self, x, y):
        self.x = x
        self.y = y

p = Point(1, 2)
p.z = 3  # AttributeError! Can't add new attributes
```

## Memory Savings

```python
# With __dict__ (default): ~104 bytes per instance
# With __slots__: ~56 bytes per instance

# For millions of objects, this matters!
```

# Properties

Properties allow controlled access to attributes.

## Basic Property

```python
class Circle:
    def __init__(self, radius):
        self._radius = radius
    
    @property
    def radius(self):
        return self._radius
    
    @radius.setter
    def radius(self, value):
        if value < 0:
            raise ValueError("Radius must be positive")
        self._radius = value
    
    @property
    def area(self):  # Computed property (read-only)
        return 3.14159 * self._radius ** 2

c = Circle(5)
print(c.radius)  # 5 (getter)
c.radius = 10    # setter
print(c.area)    # 314.159 (computed)
```

## Property Decorator Chain

```python
class Temperature:
    def __init__(self, celsius=0):
        self._celsius = celsius
    
    @property
    def celsius(self):
        return self._celsius
    
    @celsius.setter
    def celsius(self, value):
        self._celsius = value
    
    @property
    def fahrenheit(self):
        return self._celsius * 9/5 + 32
    
    @fahrenheit.setter
    def fahrenheit(self, value):
        self._celsius = (value - 32) * 5/9
```

## Code Examples

**Slots with dataclass**

```python
# Combining slots with dataclass (Python 3.10+)
from dataclasses import dataclass

@dataclass(slots=True)
class Pixel:
    x: int
    y: int
    color: str

# Memory-efficient and has all dataclass conveniences
```


---

> 📘 *This lesson is part of the [Python Fundamentals: Modern 3.15 Edition](https://stanza.dev/courses/python) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*