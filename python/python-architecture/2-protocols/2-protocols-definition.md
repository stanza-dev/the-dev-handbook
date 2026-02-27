---
source_course: "python-architecture"
source_lesson: "python-architecture-protocols-definition"
---

# Defining and Using Protocols

## Introduction

Now that you understand what Protocols are, it is time to learn how to define them in practice. A Protocol can describe methods, properties, class variables, and even combine multiple Protocols into one. Mastering these building blocks gives you a powerful toolkit for expressing interfaces without imposing inheritance constraints.

## Key Concepts

- **Method Protocol**: A Protocol that declares one or more method signatures that conforming classes must implement.
- **Property Protocol**: A Protocol that requires conforming classes to expose a specific property with a given type.
- **Protocol composition**: Combining multiple Protocols by having a new Protocol inherit from several existing ones, creating a richer interface from smaller pieces.

## Real World Context

Consider a rendering engine that draws shapes to a canvas. You want every shape to have a `draw()` method, but shapes come from different libraries, each with its own class hierarchy. By defining a `Drawable` Protocol, you can accept any shape that has the right method, regardless of where it comes from. As your engine grows, you might also need shapes that report their name or combine drawing with serialization -- Protocol composition handles all of this cleanly.

## Deep Dive

The simplest Protocol declares a single method. Any class with a matching method signature satisfies it automatically. Here is a `Drawable` Protocol and two classes that conform to it without any inheritance:

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

Both `Circle` and `Square` satisfy `Drawable` because they each have a `draw(self) -> None` method. The type checker verifies this at analysis time, and no runtime inheritance is involved.

Protocols can also require properties. The following `Named` Protocol demands a read-only `name` property. Any class that exposes a `name` property returning `str` will satisfy it:

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

The `Person` class satisfies `Named` because it provides a `name` property that returns `str`. Notice that the internal storage (`self._name`) is an implementation detail that the Protocol does not care about.

You can also require class-level attributes. The following `Versioned` Protocol ensures that any conforming class has a `version` string:

```python
class Versioned(Protocol):
    version: str  # Must have this attribute

class MyAPI:
    version = "1.0.0"
```

`MyAPI` satisfies `Versioned` because it has a `version` attribute of type `str`. This pattern is useful for configuration objects, metadata carriers, and versioned contracts.

Finally, you can compose Protocols by combining them through multiple inheritance. This lets you build richer interfaces from small, focused pieces:

```python
class Reader(Protocol):
    def read(self) -> str: ...

class Writer(Protocol):
    def write(self, data: str) -> None: ...

class ReadWriter(Reader, Writer, Protocol):
    pass  # Combines both
```

A class that has both `read()` and `write()` methods will satisfy `ReadWriter`. This compositional approach is the Protocol equivalent of interface segregation: define small Protocols and combine them only when you need the union.

## Common Pitfalls

- **Forgetting `Protocol` in composed Protocols**: When combining `Reader` and `Writer`, the composed class must also list `Protocol` as a base (i.e., `class ReadWriter(Reader, Writer, Protocol)`). Without it, you create a regular class, not a Protocol.
- **Using mutable properties in Protocols**: If your Protocol declares a property, the conforming class must use `@property` as well. A plain instance attribute may not always match the property contract in all type checkers.
- **Overloading Protocols with too many members**: A Protocol with ten methods is hard to satisfy and defeats the purpose of structural typing. Split it into smaller Protocols and compose them when needed.

## Best Practices

- **Start with single-method Protocols**: The simplest Protocols are the most reusable. You can always compose them later.
- **Document Protocol methods with docstrings**: Even though Protocol methods have no real body, a docstring communicates intent to implementers and shows up in IDE tooltips.
- **Prefer composition over monolithic Protocols**: Instead of one large Protocol, define several small ones and combine them. This follows the Interface Segregation Principle.

## Summary

- A basic Protocol declares method signatures that conforming classes must match.
- Protocols can require properties and class-level attributes, not just methods.
- Protocol composition lets you combine small Protocols into richer interfaces via multiple inheritance.
- Always include `Protocol` as a base class when composing Protocols together.
- Keep Protocols focused and use composition to build complexity incrementally.

## Code Examples

**Generic protocols**

```python
from typing import Protocol, Iterator

class Iterable[T](Protocol):
    def __iter__(self) -> Iterator[T]: ...

class Container[T](Protocol):
    def __contains__(self, item: T) -> bool: ...

# Generic protocol using PEP 695 syntax
class Collection[T](Iterable[T], Container[T], Protocol):
    def __len__(self) -> int: ...

def process[T](items: Collection[T]) -> int:
    return len(items)
```


## Resources

- [typing — Protocol](https://docs.python.org/3.14/library/typing.html#typing.Protocol) — Official reference for defining Protocols with methods, properties, and composition

---

> 📘 *This lesson is part of the [Python Architecture: Patterns & Type System](https://stanza.dev/courses/python-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*