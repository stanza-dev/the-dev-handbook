---
source_course: "python"
source_lesson: "python-slots-properties"
---

# Slots & Properties

## Introduction
`__slots__` and properties are two mechanisms for controlling how attributes are stored and accessed on Python objects. Slots optimize memory; properties add validation and computed values behind a clean attribute-access syntax.

## Key Concepts
- **`__slots__`**: A class-level tuple that restricts instances to a fixed set of attributes, eliminating `__dict__` and saving memory.
- **`@property`**: A decorator that turns a method into a read-only attribute accessor.
- **Setter / deleter**: Additional decorators (`@attr.setter`, `@attr.deleter`) that add write and delete support to a property.
- **Computed property**: A read-only property that derives its value from other attributes.

## Real World Context
When you create millions of small objects (e.g., game entities, pixel data, or sensor readings), `__slots__` can cut memory usage in half. Properties are used throughout ORMs and form libraries to validate data on assignment -- for example, ensuring a user's age is non-negative or a price is formatted correctly before it is stored.

## Deep Dive

### __slots__

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

#### Memory Savings

```python
# With __dict__ (default): ~104 bytes per instance
# With __slots__: ~56 bytes per instance

# For millions of objects, this matters!
```

### Properties

Properties allow controlled access to attributes.

#### Basic Property

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

#### Property Decorator Chain

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

## Common Pitfalls
1. **Forgetting to include all attributes in `__slots__`** -- Any attribute not listed in `__slots__` cannot be set, which causes `AttributeError`. If you inherit from a class without slots, you still get a `__dict__` from the parent.
2. **Defining a setter without a getter** -- You must define the `@property` getter first. The setter uses `@property_name.setter`, which does not exist until the getter is defined.
3. **Expensive computation in a property without caching** -- If a property performs heavy work, it runs on every access. Use `functools.cached_property` (Python 3.8+) to compute once and cache the result.

## Best Practices
1. **Use `@dataclass(slots=True)` for the best of both worlds** -- You get auto-generated methods and memory-efficient slots without writing `__slots__` manually.
2. **Use properties for validation at the boundary** -- Validate data when it is set, not when it is used. This catches errors early.

## Summary
- `__slots__` restricts instance attributes to a fixed set and reduces memory usage significantly.
- Properties provide getter/setter/deleter access behind clean attribute syntax.
- Use computed properties for derived values like `area` from `radius`.
- Consider `functools.cached_property` for expensive computations.
- Combine slots with dataclasses via `@dataclass(slots=True)` for convenience and efficiency.

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


## Resources

- [__slots__](https://docs.python.org/3.14/reference/datamodel.html#slots) — Official Python 3.14 reference for __slots__ and memory-efficient attribute storage

---

> 📘 *This lesson is part of the [Python Fundamentals: Modern 3.14 Edition](https://stanza.dev/courses/python) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*