---
source_course: "python-architecture"
source_lesson: "python-architecture-protocols-definition"
---

# Defining Protocols

## Basic Protocol

```python
from typing import Protocol

class Drawable(Protocol):
    def draw(self) -> None:
        """Draw the object."""
        ...

def render(shape: Drawable) -> None:
    shape.draw()

class Circle:
    def draw(self) -> None:
        print("O")

class Square:
    def draw(self) -> None:
        print("[]")

render(Circle())  # Valid
render(Square())  # Valid
```

## Protocols with Properties

```python
class Named(Protocol):
    @property
    def name(self) -> str:
        ...

class Person:
    def __init__(self, name: str):
        self._name = name
    
    @property
    def name(self) -> str:
        return self._name

def greet(obj: Named) -> str:
    return f"Hello, {obj.name}"
```

## Protocols with Class Variables

```python
class Versioned(Protocol):
    version: str  # Must have this attribute

class MyAPI:
    version = "1.0.0"
```

## Combining Protocols

```python
class Reader(Protocol):
    def read(self) -> str: ...

class Writer(Protocol):
    def write(self, data: str) -> None: ...

class ReadWriter(Reader, Writer, Protocol):
    pass  # Combines both
```

## Code Examples

**Generic protocols**

```python
from typing import Protocol, Iterator

class Iterable(Protocol[T]):
    def __iter__(self) -> Iterator[T]: ...

class Container(Protocol[T]):
    def __contains__(self, item: T) -> bool: ...

# Generic protocol
class Collection(Iterable[T], Container[T], Protocol):
    def __len__(self) -> int: ...

def process[T](items: Collection[T]) -> int:
    return len(items)
```


---

> 📘 *This lesson is part of the [Python Architecture: Patterns & Type System](https://stanza.dev/courses/python-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*