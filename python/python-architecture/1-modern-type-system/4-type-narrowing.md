---
source_course: "python-architecture"
source_lesson: "python-architecture-type-narrowing"
---

# Type Narrowing & Guards

## Introduction

Union types like `int | str | None` are powerful, but at some point you need to do something specific with the concrete type inside the union. Type narrowing is the mechanism by which a type checker deduces a more specific type after a conditional check. This lesson covers every narrowing technique available in Python 3.14, from basic `isinstance` checks to custom type guards and pattern matching.

## Key Concepts

- **Type narrowing**: The process by which a type checker refines a broad type (e.g., `int | str`) to a more specific one (e.g., `int`) inside a conditional branch.
- **TypeGuard**: A special return-type annotation for boolean functions that tells the checker to narrow the input type when the function returns `True`.
- **TypeIs**: A stricter alternative to `TypeGuard` (Python 3.13+) that narrows in both the `True` and `False` branches and preserves type compatibility.
- **Pattern matching narrowing**: Structural pattern matching (`match`/`case`) that narrows types based on the matched pattern.

## Real World Context

You receive a webhook payload that could be a `dict`, a `list`, or a raw `str`. You need to route it to the correct handler. Without narrowing, the type checker would complain about calling `.keys()` on `str` or `.upper()` on `dict`. By using `isinstance` checks or pattern matching, you prove to both the checker and future readers that each branch handles exactly the right type -- no casting, no `# type: ignore`, no runtime surprises.

## Deep Dive

The most common narrowing technique is an `isinstance` check. After the check, the type checker knows the variable's type within that branch.

```python
def process(value: int | str) -> str:
    if isinstance(value, int):
        return str(value * 2)  # Narrowed to int
    return value.upper()  # Narrowed to str
```

Inside the `if` block, `value` is `int`. In the `else` branch (or after the `if` returns), the checker eliminates `int` and knows `value` is `str`.

None checks work the same way. An `is None` or `is not None` comparison narrows `X | None` to just `None` or just `X`.

```python
def greet(name: str | None) -> str:
    if name is None:
        return "Hello, stranger"
    return f"Hello, {name}"  # Narrowed to str

# Or with assert
assert name is not None
print(name.upper())  # Narrowed
```

The `assert` approach is concise but will raise `AssertionError` at runtime if the assertion fails, which can be useful as a defensive check.

For more complex validation logic that does not fit a single `isinstance`, you can write a custom type guard function using `TypeGuard`. The checker trusts the function's return value to narrow the input type.

```python
from typing import TypeGuard

def is_string_list(val: list[object]) -> TypeGuard[list[str]]:
    return all(isinstance(x, str) for x in val)

def process(items: list[object]) -> None:
    if is_string_list(items):
        # items is now list[str]
        print(items[0].upper())
```

When `is_string_list` returns `True`, the checker narrows `items` from `list[object]` to `list[str]` inside the `if` block. Note that `TypeGuard` only narrows in the `True` branch.

Python 3.13 introduced `TypeIs`, which is a stricter and usually preferable alternative. Unlike `TypeGuard`, `TypeIs` narrows in both the `True` and `False` branches and requires that the narrowed type be compatible with the input type.

```python
from typing import TypeIs

def is_str(val: object) -> TypeIs[str]:
    return isinstance(val, str)

def process(val: object) -> None:
    if is_str(val):
        print(val.upper())  # val is str
```

In the `else` branch of a `TypeIs` guard, the checker knows `val` is not `str`, which enables further narrowing downstream.

Python's structural pattern matching also narrows types. Each `case` branch refines the type of the matched variable based on the pattern.

```python
def handle(data: dict | list | str) -> None:
    match data:
        case dict():
            print(data.keys())  # Narrowed to dict
        case list():
            print(len(data))  # Narrowed to list
        case str():
            print(data.upper())  # Narrowed to str
```

Pattern matching is especially readable when handling multiple branches, and the checker exhaustiveness analysis can warn you if a union member is unhandled.

## Common Pitfalls

- **Writing a `TypeGuard` that lies.** The checker trusts your guard function unconditionally. If `is_string_list` returns `True` for a list containing integers, you will get silent type errors at runtime. Always ensure the guard's logic matches its declared narrowing.
- **Forgetting that `TypeGuard` only narrows in the `True` branch.** In the `else` branch, the original broad type remains. If you need narrowing in both branches, use `TypeIs` instead.
- **Using `assert` for narrowing in production code.** Python's `-O` flag strips assertions entirely. If your narrowing logic doubles as a runtime safety check, use an explicit `if` with a raised exception instead.

## Best Practices

- **Prefer `TypeIs` over `TypeGuard` in Python 3.13+.** `TypeIs` is safer because it narrows both branches and enforces type compatibility, reducing the chance of an incorrect guard.
- **Use pattern matching for multi-branch narrowing.** When you need to handle three or more union members, `match`/`case` is cleaner and more maintainable than a chain of `isinstance` checks.
- **Keep type guard functions simple and testable.** A guard should perform one clear validation. Complex guards with side effects are harder to trust and maintain.

## Summary

- `isinstance` checks and `is None` comparisons are the most common narrowing triggers, automatically recognized by type checkers.
- `TypeGuard` lets you write custom boolean functions that narrow a parameter's type in the `True` branch.
- `TypeIs` (Python 3.13+) is a stricter alternative that narrows in both branches and enforces type compatibility.
- Pattern matching (`match`/`case`) narrows types based on structural patterns and supports exhaustiveness checking.
- Always ensure custom type guards are correct -- the checker trusts them unconditionally.

## Code Examples

**TypeGuard for validation**

```python
from typing import TypeGuard, Any

def is_user_dict(val: Any) -> TypeGuard[dict[str, str]]:
    """Check if value is a user dictionary."""
    return (
        isinstance(val, dict) and
        isinstance(val.get("name"), str) and
        isinstance(val.get("email"), str)
    )

def process_user(data: Any) -> str:
    if is_user_dict(data):
        # Type checker knows data is dict[str, str]
        return f"User: {data['name']} <{data['email']}>"
    raise ValueError("Invalid user data")
```


## Resources

- [PEP 742 — Narrowing types with TypeIs](https://peps.python.org/pep-0742/) — The PEP introducing TypeIs for bidirectional narrowing
- [typing — TypeGuard](https://docs.python.org/3.14/library/typing.html#typing.TypeGuard) — Official docs for TypeGuard

---

> 📘 *This lesson is part of the [Python Architecture: Patterns & Type System](https://stanza.dev/courses/python-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*