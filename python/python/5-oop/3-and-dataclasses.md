---
source_course: "python"
source_lesson: "python-classes-and-dataclasses"
---

# Data Classes

## Introduction
Data classes eliminate the boilerplate of writing `__init__`, `__repr__`, and `__eq__` for classes that primarily hold data. With a single decorator, Python generates these methods for you automatically. This lesson covers the `@dataclass` decorator, its options, field configuration, and inheritance.

## Key Concepts
- **`@dataclass`**: A decorator that auto-generates `__init__`, `__repr__`, and `__eq__` from annotated class attributes.
- **`frozen=True`**: Makes instances immutable (read-only after creation).
- **`slots=True`**: Uses `__slots__` for memory-efficient attribute storage (Python 3.10+).
- **`field()`**: Configures individual attributes with defaults, factories, and metadata.
- **`__post_init__`**: A hook that runs after `__init__` for computed attributes.

## Real World Context
Data classes are the go-to for DTOs, configuration objects, API response models, and database rows. Pydantic v2 uses a dataclass-like API for validation. Choosing `frozen=True` gives you hashable, immutable objects that can be used as dict keys or set members -- ideal for caching and deduplication patterns.

## Deep Dive

### Basic Data Class

```python
from dataclasses import dataclass

@dataclass
class User:
    id: int
    name: str
    email: str
    active: bool = True  # Default value

# Automatically generates:
# - __init__(self, id, name, email, active=True)
# - __repr__(self)
# - __eq__(self, other)
```

### Data Class Options

```python
@dataclass(frozen=True)  # Immutable
class Point:
    x: float
    y: float

@dataclass(order=True)   # Adds comparison methods
class Person:
    name: str
    age: int

@dataclass(slots=True)   # Use __slots__ (3.10+)
class Efficient:
    value: int
```

### Field Options

```python
from dataclasses import dataclass, field

@dataclass
class Config:
    name: str
    tags: list = field(default_factory=list)  # Mutable default
    _internal: str = field(repr=False)  # Exclude from repr
    computed: str = field(init=False)   # Not in __init__
    
    def __post_init__(self):
        self.computed = self.name.upper()
```

### Inheritance with Data Classes

```python
@dataclass
class Animal:
    name: str

@dataclass
class Dog(Animal):
    breed: str

dog = Dog(name="Buddy", breed="Labrador")
```

## Common Pitfalls
1. **Using a mutable default directly** -- `tags: list = []` is a bug: all instances share the same list. Always use `field(default_factory=list)`.
2. **Expecting `__hash__` to be generated automatically** -- When `eq=True` (the default), `__hash__` is set to `None` to prevent mutable objects from being used as dict keys. Use `frozen=True` if you need hashability.
3. **Putting fields with defaults before fields without** -- This causes a TypeError. Non-default fields must come first, or use `field(init=False)` for computed defaults.

## Best Practices
1. **Use `frozen=True` for value objects** -- Immutable data classes are safer to pass around and can be used in sets and as dict keys.
2. **Combine `slots=True` with `frozen=True` for maximum efficiency** -- This gives you immutable, memory-efficient objects ideal for large collections.

## Summary
- `@dataclass` auto-generates `__init__`, `__repr__`, and `__eq__` from type-annotated attributes.
- Use `frozen=True` for immutable, hashable objects and `slots=True` for memory savings.
- Always use `field(default_factory=...)` for mutable defaults like lists and dicts.
- `__post_init__` lets you compute derived attributes after initialization.
- Data classes support inheritance; fields from parent classes come first in `__init__`.

## Code Examples

**Data class utilities**

```python
from dataclasses import dataclass, asdict, astuple

@dataclass
class Product:
    name: str
    price: float
    quantity: int = 0

p = Product("Widget", 9.99, 100)

# Convert to dict/tuple
print(asdict(p))   # {'name': 'Widget', 'price': 9.99, 'quantity': 100}
print(astuple(p))  # ('Widget', 9.99, 100)

# Immutable dataclass
@dataclass(frozen=True)
class Coordinate:
    lat: float
    lon: float

c = Coordinate(40.7, -74.0)
# c.lat = 0  # FrozenInstanceError!
```


## Resources

- [dataclasses Module](https://docs.python.org/3.14/library/dataclasses.html) — Official Python 3.14 reference for the dataclasses module and decorator

---

> 📘 *This lesson is part of the [Python Fundamentals: Modern 3.14 Edition](https://stanza.dev/courses/python) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*