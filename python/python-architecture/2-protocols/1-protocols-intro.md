---
source_course: "python-architecture"
source_lesson: "python-architecture-protocols-intro"
---

# Introduction to Protocols

## Introduction

Python has always embraced duck typing: if an object has the right methods, it works, no inheritance required. But duck typing has a blind spot -- type checkers and IDEs cannot verify that the right methods actually exist until runtime. Protocols bridge this gap, letting you describe the shape of an object so that static tools can catch mistakes before your code ever runs.

## Key Concepts

- **Duck typing**: A convention where an object's suitability is determined by the presence of certain methods and properties, rather than its actual type or inheritance hierarchy.
- **Structural subtyping**: A formal type-checking strategy where a type is considered compatible if it has the required structure (methods, attributes), even without explicit inheritance.
- **Protocol**: A special class from `typing` that defines a structural interface. Any class matching that structure satisfies the Protocol automatically.

## Real World Context

Imagine you are building a plugin system where third-party developers provide their own handler classes. You cannot force them to inherit from your base class, but you need your type checker and IDE to verify that every plugin has a `handle()` method. Protocols let you write a formal contract that any conforming class will satisfy without coupling your code to a specific inheritance tree.

## Deep Dive

Python's duck typing philosophy is often summarized as: "If it walks like a duck and quacks like a duck, it's a duck." In practice, this means you can write functions that accept any object with the right methods.

Here is a classic duck-typing example where any object with `read()` and `write()` methods will work:

```python
def process(obj):
    obj.read()  # Works with any object that has read()
    obj.write(data)  # Works with any object that has write()
```

This is flexible, but the type checker has no idea what `obj` should look like. There is no autocomplete, no error detection, and no documentation of the expected interface.

Protocols solve this by giving duck typing a formal definition. You create a class that inherits from `Protocol` and declare the methods you expect. The following example defines a `Readable` protocol and uses it as a type hint:

```python
from typing import Protocol

class Readable(Protocol):
    def read(self) -> str:
        ...

def process(obj: Readable) -> None:
    content = obj.read()  # Type checker knows this is str
```

Now the type checker understands that `obj` must have a `read()` method returning `str`, and your IDE can provide autocomplete and inline documentation.

The crucial insight is **structural subtyping**: a class satisfies a Protocol simply by having the required methods. It does not need to inherit from the Protocol at all. The following `File` class satisfies `Readable` even though it has no relationship to it:

```python
# This works even though File doesn't inherit from Readable!
class File:
    def read(self) -> str:
        return "content"

process(File())  # Valid!
```

The type checker sees that `File` has a `read() -> str` method and considers it a valid `Readable`. This is the power of Protocols: you get the safety of static typing with the flexibility of duck typing.

## Common Pitfalls

- **Assuming Protocols require inheritance**: The entire point of a Protocol is structural subtyping. If you find yourself writing `class MyClass(Readable):`, you are using it like an ABC. Drop the inheritance; if `MyClass` has the right methods, it already satisfies the Protocol.
- **Forgetting the ellipsis body**: Protocol method bodies should contain `...` (Ellipsis) to signal they are abstract declarations, not implementations. Writing `pass` works but is less idiomatic.
- **Mismatched return types**: If your Protocol declares `def read(self) -> str`, but your class has `def read(self) -> bytes`, the type checker will reject it. Return types must be compatible.

## Best Practices

- **Define Protocols near the consumer, not the implementer**: The function that needs the interface should own the Protocol definition. This keeps dependencies pointing inward.
- **Keep Protocols small and focused**: A Protocol with one or two methods is easier to satisfy and compose than a monolithic one with ten methods.
- **Use Protocols for boundaries**: They shine at API boundaries, plugin systems, and anywhere you interact with code you do not control.

## Summary

- Duck typing is powerful but invisible to static analysis tools.
- Protocols formalize duck typing by declaring the methods an object must have.
- Structural subtyping means classes satisfy a Protocol by shape, not by inheritance.
- Type checkers and IDEs can now verify correctness and provide autocomplete for duck-typed code.
- Keep Protocols small, focused, and owned by the code that consumes them.

## Code Examples

**Protocol as type constraint**

```python
from typing import Protocol

class Comparable(Protocol):
    def __lt__(self, other) -> bool: ...
    def __eq__(self, other) -> bool: ...

def find_min[T: Comparable](items: list[T]) -> T:
    return min(items)

# Works with any type that has __lt__ and __eq__
find_min([3, 1, 2])  # int has these methods
find_min(["c", "a", "b"])  # str has these methods
```


## Resources

- [PEP 544 — Protocols: Structural subtyping](https://peps.python.org/pep-0544/) — The PEP that introduced Protocol to Python
- [typing — Protocol](https://docs.python.org/3.14/library/typing.html#typing.Protocol) — Official Protocol class documentation

---

> 📘 *This lesson is part of the [Python Architecture: Patterns & Type System](https://stanza.dev/courses/python-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*