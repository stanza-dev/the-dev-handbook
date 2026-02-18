---
source_course: "python-architecture"
source_lesson: "python-architecture-protocols-intro"
---

# Duck Typing and Protocols

## The Python Way: Duck Typing

"If it walks like a duck and quacks like a duck, it's a duck."

```python
def process(obj):
    obj.read()  # Works with any object that has read()
    obj.write(data)  # Works with any object that has write()
```

## The Problem

Duck typing is flexible but:
- Type checkers can't verify correctness
- IDEs can't provide autocomplete
- Documentation is implicit

## The Solution: Protocols

```python
from typing import Protocol

class Readable(Protocol):
    def read(self) -> str:
        ...

def process(obj: Readable) -> None:
    content = obj.read()  # Type checker knows this is str
```

## Key Concept

Protocols use **structural subtyping**: a class satisfies a Protocol if it has the required methods, *without inheriting from it*.

```python
# This works even though File doesn't inherit from Readable!
class File:
    def read(self) -> str:
        return "content"

process(File())  # Valid!
```

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


---

> 📘 *This lesson is part of the [Python Architecture: Patterns & Type System](https://stanza.dev/courses/python-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*