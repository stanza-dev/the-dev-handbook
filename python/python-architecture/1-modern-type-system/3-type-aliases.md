---
source_course: "python-architecture"
source_lesson: "python-architecture-generics-type-aliases"
---

# Modern Generics (Python 3.12+)

PEP 695 introduced a cleaner syntax for generics.

## Type Aliases

```python
# Old way
from typing import TypeAlias
Vector: TypeAlias = list[float]

# New way (3.12+)
type Vector = list[float]
type Result[T] = T | None
type Handler[T, R] = Callable[[T], R]
```

## Generic Functions

```python
# Old way
from typing import TypeVar
T = TypeVar('T')
def first(items: list[T]) -> T:
    return items[0]

# New way (3.12+)
def first[T](items: list[T]) -> T:
    return items[0]

def pair[T, U](a: T, b: U) -> tuple[T, U]:
    return (a, b)
```

## Generic Classes

```python
# Old way
from typing import Generic, TypeVar
T = TypeVar('T')
class Box(Generic[T]):
    def __init__(self, value: T): ...

# New way (3.12+)
class Box[T]:
    def __init__(self, value: T) -> None:
        self.value = value
    
    def get(self) -> T:
        return self.value

box: Box[int] = Box(42)
```

## Bounded Type Variables

```python
# Constrained to specific types
def add[T: (int, float)](a: T, b: T) -> T:
    return a + b

# Upper bound
from collections.abc import Sequence
def total[T: Sequence](items: T) -> int:
    return len(items)
```

## Code Examples

**Generic Stack implementation**

```python
# Complete example: Generic Stack
class Stack[T]:
    def __init__(self) -> None:
        self._items: list[T] = []
    
    def push(self, item: T) -> None:
        self._items.append(item)
    
    def pop(self) -> T:
        return self._items.pop()
    
    def peek(self) -> T | None:
        return self._items[-1] if self._items else None
    
    def is_empty(self) -> bool:
        return len(self._items) == 0

# Type-safe usage
int_stack: Stack[int] = Stack()
int_stack.push(1)
int_stack.push(2)
value: int = int_stack.pop()  # Type checker knows this is int
```


## Resources

- [PEP 695](https://peps.python.org/pep-0695/) — Type Parameter Syntax

---

> 📘 *This lesson is part of the [Python Architecture: Patterns & Type System](https://stanza.dev/courses/python-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*