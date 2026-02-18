---
source_course: "python-architecture"
source_lesson: "python-architecture-type-hints-basics"
---

# Type Hints in Python

Type hints are optional annotations that help with code documentation and static analysis.

## Basic Syntax

```python
def greet(name: str) -> str:
    return f"Hello, {name}"

age: int = 25
names: list[str] = ["Alice", "Bob"]
```

## Built-in Types

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

## Optional and Union

```python
from typing import Optional

# Optional = Union with None
def find(name: str) -> Optional[int]:
    return None

# Modern union syntax (3.10+)
def process(data: int | str | None) -> str:
    return str(data)
```

## Type Checkers

Type hints don't affect runtime - use static tools:

```bash
# mypy
pip install mypy
mypy your_code.py

# pyright (faster)
pip install pyright
pyright your_code.py
```

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


---

> 📘 *This lesson is part of the [Python Architecture: Patterns & Type System](https://stanza.dev/courses/python-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*