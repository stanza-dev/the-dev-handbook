---
source_course: "python-architecture"
source_lesson: "python-architecture-advanced-types"
---

# Advanced Type Annotations

## Introduction

Basic type hints cover primitives and collections, but real-world Python code passes around callbacks, restricts values to fixed sets of strings, and models complex dictionary shapes. This lesson introduces the advanced annotation tools -- `Callable`, `Literal`, `TypedDict`, `Final`, `ClassVar`, and `Self` -- that let you express these richer contracts precisely.

## Key Concepts

- **Callable**: A type that describes a function's parameter types and return type, so you can safely pass functions as arguments.
- **Literal**: A type that restricts a value to one of a fixed set of literal constants, catching invalid strings or numbers at check time.
- **TypedDict**: A way to give a dictionary a fixed schema with per-key types, bridging the gap between dicts and dataclasses.
- **Final**: An annotation that prevents a name from being reassigned after its initial binding.
- **Self**: A return-type annotation that tells the checker a method returns the same type as the enclosing class, even in subclasses.

## Real World Context

Consider a plugin system where users register callback functions, or a configuration layer that accepts only `"development"`, `"staging"`, or `"production"` as environment names. Without `Callable` and `Literal`, these constraints live only in docstrings and are silently violated. Advanced type annotations move those constraints into the type system where every editor and CI pipeline can enforce them.

## Deep Dive

When a function accepts another function as a parameter, you need `Callable` to describe the expected signature. The first element is a list of parameter types; the second is the return type.

```python
from typing import Callable

# Function that takes str and int, returns bool
Predicate = Callable[[str, int], bool]

def apply(func: Callable[[int], int], value: int) -> int:
    return func(value)
```

With this annotation, passing a function that returns `str` instead of `int` is immediately flagged by the type checker.

`Literal` restricts a value to an exact set of constants. This is far more precise than annotating a parameter as `str` when only a handful of values are valid.

```python
from typing import Literal

Mode = Literal["read", "write", "append"]

def open_file(path: str, mode: Mode) -> None:
    pass

open_file("x.txt", "read")   # OK
open_file("x.txt", "delete")  # Type error!
```

The checker will reject `"delete"` at analysis time, which is much safer than catching the typo through a runtime `ValueError`.

`TypedDict` gives a dictionary a fixed set of keys, each with its own type. This is invaluable when working with JSON payloads or legacy code that passes dictionaries instead of dataclasses.

```python
from typing import TypedDict

class UserDict(TypedDict):
    name: str
    age: int
    email: str  # All required by default

class PartialUser(TypedDict, total=False):
    name: str
    age: int  # All optional

user: UserDict = {"name": "Alice", "age": 30, "email": "a@b.com"}
```

Setting `total=False` makes every key optional, which is useful for update payloads where the caller may supply only a subset of fields.

`Final` marks a binding as immutable after assignment -- the type checker will report an error if any code tries to reassign it. `ClassVar` distinguishes class-level attributes from instance attributes inside a class body.

```python
from typing import Final, ClassVar

MAX_SIZE: Final = 100  # Cannot be reassigned

class Config:
    instance_count: ClassVar[int] = 0  # Class variable
    name: str  # Instance variable
```

Using `Final` for module-level constants protects you from accidental mutation, and `ClassVar` prevents dataclass-related tools from treating class variables as constructor parameters.

`Self` solves the problem of annotating fluent (builder-style) methods. Before `Self`, you had to use complicated `TypeVar` tricks to ensure subclasses returned the correct type.

```python
from typing import Self

class Builder:
    def set_name(self, name: str) -> Self:
        self.name = name
        return self  # Returns Self for chaining
```

Because the return type is `Self`, a subclass inheriting `set_name` will correctly report the subclass type, not the parent.

## Common Pitfalls

- **Using `Callable` without parameter types.** `Callable[..., int]` with the ellipsis accepts any arguments, which defeats the purpose. Spell out parameter types when you know them.
- **Forgetting `total=False` for partial update dicts.** If your endpoint accepts a PATCH payload with optional fields, a `TypedDict` with the default `total=True` will require every key and reject valid requests.
- **Returning `self` without `Self` in a class hierarchy.** Annotating the return as the base class type causes subclass instances to lose their specific type information. Always use `Self` for fluent return values.

## Best Practices

- **Use `Literal` over bare `str` for mode/status/action parameters.** This eliminates an entire class of invalid-argument bugs.
- **Prefer `TypedDict` for external data shapes (JSON APIs, config files)** and dataclasses for internal domain objects. This makes the boundary between validated and unvalidated data explicit.
- **Mark true constants with `Final`** so that refactoring tools and checkers can confidently inline or protect them.

## Summary

- `Callable[[ParamTypes], ReturnType]` annotates higher-order function parameters with full signature information.
- `Literal["a", "b"]` restricts values to a fixed set of constants, catching typos at analysis time.
- `TypedDict` gives dictionaries per-key types; use `total=False` for optional-key variants.
- `Final` prevents reassignment; `ClassVar` distinguishes class-level from instance-level attributes.
- `Self` enables correctly typed fluent/builder patterns across inheritance hierarchies.

## Code Examples

**TypedDict with Literal**

```python
from typing import TypedDict, Literal, Required, NotRequired

# TypedDict with mixed requirements (3.11+)
class APIResponse(TypedDict):
    status: Required[Literal["success", "error"]]
    data: NotRequired[dict]
    error: NotRequired[str]

def handle_response(response: APIResponse) -> None:
    if response["status"] == "success":
        print(response.get("data", {}))
    else:
        print(response.get("error", "Unknown error"))
```


## Resources

- [typing — Support for type hints](https://docs.python.org/3.14/library/typing.html) — Official reference for Callable, Literal, TypedDict, Final, and Self
- [PEP 586 — Literal Types](https://peps.python.org/pep-0586/) — The PEP that introduced Literal types

---

> 📘 *This lesson is part of the [Python Architecture: Patterns & Type System](https://stanza.dev/courses/python-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*