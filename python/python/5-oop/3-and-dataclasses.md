---
source_course: "python"
source_lesson: "python-classes-and-dataclasses"
---

# Data Classes

Data classes reduce boilerplate for classes that primarily store data.

## Basic Data Class

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

## Data Class Options

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

## Field Options

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

## Inheritance with Data Classes

```python
@dataclass
class Animal:
    name: str

@dataclass
class Dog(Animal):
    breed: str

dog = Dog(name="Buddy", breed="Labrador")
```

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


---

> 📘 *This lesson is part of the [Python Fundamentals: Modern 3.15 Edition](https://stanza.dev/courses/python) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*