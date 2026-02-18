---
source_course: "python-architecture"
source_lesson: "python-architecture-type-narrowing"
---

# Type Narrowing

Type checkers can narrow types based on control flow.

## isinstance Narrowing

```python
def process(value: int | str) -> str:
    if isinstance(value, int):
        return str(value * 2)  # Narrowed to int
    return value.upper()  # Narrowed to str
```

## None Checks

```python
def greet(name: str | None) -> str:
    if name is None:
        return "Hello, stranger"
    return f"Hello, {name}"  # Narrowed to str

# Or with assert
assert name is not None
print(name.upper())  # Narrowed
```

## TypeGuard (3.10+)

```python
from typing import TypeGuard

def is_string_list(val: list[object]) -> TypeGuard[list[str]]:
    return all(isinstance(x, str) for x in val)

def process(items: list[object]) -> None:
    if is_string_list(items):
        # items is now list[str]
        print(items[0].upper())
```

## TypeIs (3.13+)

```python
from typing import TypeIs

def is_str(val: object) -> TypeIs[str]:
    return isinstance(val, str)

def process(val: object) -> None:
    if is_str(val):
        print(val.upper())  # val is str
```

## Pattern Matching Type Narrowing

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


---

> 📘 *This lesson is part of the [Python Architecture: Patterns & Type System](https://stanza.dev/courses/python-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*