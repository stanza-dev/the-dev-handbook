---
source_course: "python-architecture"
source_lesson: "python-architecture-generics-type-aliases"
---

# Generics & Type Aliases (PEP 695)

## Introduction

Generics let you write functions and classes that work with any type while preserving type safety, and type aliases give complex annotations a readable name. Python 3.12 introduced PEP 695, which dramatically simplifies the syntax for both. This lesson walks you through the modern way to define generic code in Python 3.14.

## Key Concepts

- **Type alias**: A named shorthand for a complex type expression, declared with the `type` keyword in modern Python.
- **Generic function**: A function parameterized by one or more type variables, so the checker can relate input types to output types.
- **Generic class**: A class parameterized by type variables, allowing instances to be specialized (e.g., `Stack[int]` vs. `Stack[str]`).
- **Bounded type variable**: A type variable constrained to a specific set of types or an upper-bound protocol/class.

## Real World Context

You are building a data pipeline that transforms records of different shapes -- users, orders, events. Without generics, you either duplicate the transformation logic for every record type or fall back to `Any` and lose all type safety. With PEP 695 generics, you write the pipeline once, parameterize it by record type, and the checker guarantees that a `Pipeline[Order]` never accidentally processes a `User`.

## Deep Dive

Before PEP 695, creating a type alias required importing `TypeAlias` from `typing`. The new `type` statement is cleaner and supports inline type parameters.

```python
# Old way
from typing import TypeAlias
Vector: TypeAlias = list[float]

# New way (3.12+)
type Vector = list[float]
type Result[T] = T | None
type Handler[T, R] = Callable[[T], R]
```

The `type` keyword makes the alias lazy -- the right-hand side is not evaluated until the alias is used, which avoids forward-reference issues.

Generic functions previously required creating a `TypeVar` object and passing it around. PEP 695 puts the type parameter directly in the function signature, which is both shorter and harder to get wrong.

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

The bracket syntax `[T]` declares the type variable inline. Multiple type parameters are separated by commas, as shown in the `pair` function.

The same simplification applies to classes. Instead of inheriting from `Generic[T]`, you declare the type parameter directly on the class name.

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

The checker knows that `box.get()` returns `int` because the class was instantiated as `Box[int]`. This relationship is maintained throughout the class body.

Sometimes you need to restrict a type variable to certain types. Bounded type variables let you constrain `T` to a tuple of allowed types or to an upper bound.

```python
# Constrained to specific types
def add[T: (int, float)](a: T, b: T) -> T:
    return a + b

# Upper bound
from collections.abc import Sequence
def total[T: Sequence](items: T) -> int:
    return len(items)
```

With `T: (int, float)`, the function only accepts `int` or `float` arguments. With `T: Sequence`, any type that is a subtype of `Sequence` is accepted, but not unrelated types like `int`.

## Common Pitfalls

- **Mixing old and new syntax.** If you declare `T = TypeVar('T')` at module level and also use `def f[T]`, the two `T` names refer to different type variables. Pick one style and use it consistently.
- **Forgetting that type aliases are lazy.** Because `type X = ...` is lazily evaluated, referencing undefined names on the right side will not raise an error at import time -- it will fail only when the alias is resolved. Test your aliases.
- **Over-constraining type variables.** Using `T: (int,)` with a single type is pointless -- just use `int` directly. Constraints are useful only with two or more types.

## Best Practices

- **Use PEP 695 syntax for all new code.** The old `TypeVar` / `Generic` style is still supported but should be treated as legacy in Python 3.12+ projects.
- **Name type variables meaningfully** when the context is non-trivial. `T` is fine for a single parameter, but `KeyT` and `ValueT` are clearer in a mapping abstraction.
- **Combine type aliases with generics** to keep signatures readable. A `type Result[T] = T | ErrorInfo` alias is much easier to scan than inlining the union everywhere.

## Summary

- The `type` keyword creates lazy type aliases that support inline type parameters (`type Result[T] = T | None`).
- Generic functions use bracket syntax (`def f[T](x: T) -> T`) instead of module-level `TypeVar` declarations.
- Generic classes declare type parameters on the class name (`class Box[T]`) rather than inheriting from `Generic[T]`.
- Bounded type variables constrain `T` to specific types or an upper-bound class/protocol.
- PEP 695 syntax is the standard for Python 3.12+ and should be used in all new code.

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