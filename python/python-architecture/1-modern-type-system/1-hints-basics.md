---
source_course: "python-architecture"
source_lesson: "python-architecture-type-hints-basics"
---

# Type Hints Fundamentals

## Introduction

Python is dynamically typed, but since Python 3.5 you can add optional type annotations that make your code self-documenting and catch bugs before they reach production. In this lesson you will learn the foundational syntax for type hints, how to annotate variables and functions, and how to run a type checker against your code.

## Key Concepts

- **Type hint**: An annotation on a variable or function signature that declares the expected type without affecting runtime behavior.
- **Union type**: A type that accepts more than one concrete type, written with the `|` operator in modern Python.
- **Optional**: A shorthand for a union of a type with `None`, indicating that a value may be absent.
- **Type checker**: A static analysis tool (mypy, pyright) that reads type hints and reports inconsistencies before you run the program.

## Real World Context

Imagine you are maintaining an HTTP service with dozens of handler functions. Without type hints, a colleague renames a return field from `user_id` to `id` and nothing fails until a customer hits the endpoint. With type annotations and a CI type-check step, the mismatch is caught in seconds. Every professional Python team today treats type hints as a first-class part of their workflow.

## Deep Dive

The simplest type hint annotates a function's parameters and return value. The following example shows the basic colon-and-arrow syntax that every Python developer should recognize.

```python
def greet(name: str) -> str:
    return f"Hello, {name}"

age: int = 25
names: list[str] = ["Alice", "Bob"]
```

The `: str` after `name` tells readers (and tools) that `name` must be a string, while `-> str` declares the return type. Variable annotations work the same way.

Python ships with built-in generic types for all common data structures. Since Python 3.9 you can use the lowercase forms directly, and in 3.14 this is the standard convention.

```python
# Primitives
x: int = 1
y: float = 3.14
z: str = "hello"
flag: bool = True

# Collections (3.9+)
numbers: list[int] = [1, 2, 3]
coords: tuple[float, float] = (10.0, 20.0)
config: dict[str, int] = {"port": 8080}
unique: set[str] = {"a", "b"}
```

Notice how `tuple[float, float]` fixes the length and element types, whereas `list[int]` describes a variable-length sequence. This distinction is important when modeling fixed-structure records versus open-ended collections.

When a value may be absent, you can express that with `Optional` or the modern union syntax. The pipe (`|`) operator, available since Python 3.10, is now the preferred style.

```python
from typing import Optional

# Optional = Union with None
def find(name: str) -> Optional[int]:
    return None

# Modern union syntax (3.10+)
def process(data: int | str | None) -> str:
    return str(data)
```

`Optional[int]` is strictly equivalent to `int | None`. In new code, prefer the pipe syntax because it is shorter and does not require an import.

Type hints are purely informational at runtime -- Python never enforces them during execution. To actually validate your annotations, run a dedicated type checker as part of your development or CI workflow.

```bash
# mypy
pip install mypy
mypy your_code.py

# pyright (faster)
pip install pyright
pyright your_code.py
```

Both tools read the same annotations but have slightly different strictness defaults. Many teams run pyright locally for speed and mypy in CI for thorough coverage.

## Common Pitfalls

- **Using `list` without a type parameter.** Writing `def f(items: list)` is legal but nearly useless -- the checker cannot verify what the list contains. Always parameterize: `list[int]`.
- **Confusing `Optional[X]` with "the argument is optional."** `Optional[int]` means the value can be `int` or `None`; it says nothing about whether the argument has a default value.
- **Importing from `typing` when built-in generics suffice.** Since Python 3.9 you no longer need `typing.List`, `typing.Dict`, etc. Prefer the lowercase built-in forms.

## Best Practices

- **Annotate public function signatures first.** You do not need to annotate every local variable; the checker can infer those. Focus on module boundaries.
- **Enable strict mode in your type checker** (`mypy --strict` or pyright's strict config) to catch implicit `Any` types that silently weaken your guarantees.
- **Integrate the type checker into CI** so that type errors block merges, just like failing tests.

## Summary

- Type hints annotate variables and functions with expected types using `: Type` and `-> ReturnType` syntax.
- Built-in generic collections (`list[int]`, `dict[str, int]`, `tuple[float, float]`) replace the older `typing` module equivalents.
- `Optional[X]` is equivalent to `X | None`; prefer the modern pipe syntax.
- Type hints have zero runtime cost -- enforcement comes from static checkers like mypy and pyright.
- Always parameterize collection types and integrate a type checker into your CI pipeline.

## Code Examples

**Type hints in practice**

```python
from typing import Optional

def divide(a: float, b: float) -> Optional[float]:
    """Divide a by b, return None if b is zero."""
    if b == 0:
        return None
    return a / b

# Type hints are documentation + tool support
result = divide(10, 2)  # IDE knows this is float | None
if result is not None:
    print(result * 2)  # IDE knows this is safe
```


## Resources

- [typing — Support for type hints](https://docs.python.org/3.14/library/typing.html) — Official Python typing module documentation
- [mypy — Optional Static Typing](https://mypy.readthedocs.io/en/stable/) — mypy type checker documentation

---

> 📘 *This lesson is part of the [Python Architecture: Patterns & Type System](https://stanza.dev/courses/python-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*