---
source_course: "python-architecture"
source_lesson: "python-architecture-advanced-types"
---

# Advanced Typing Features

## Callable Types

```python
from typing import Callable

# Function that takes str and int, returns bool
Predicate = Callable[[str, int], bool]

def apply(func: Callable[[int], int], value: int) -> int:
    return func(value)
```

## Literal Types

```python
from typing import Literal

Mode = Literal["read", "write", "append"]

def open_file(path: str, mode: Mode) -> None:
    pass

open_file("x.txt", "read")   # OK
open_file("x.txt", "delete")  # Type error!
```

## TypedDict

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

## Final and ClassVar

```python
from typing import Final, ClassVar

MAX_SIZE: Final = 100  # Cannot be reassigned

class Config:
    instance_count: ClassVar[int] = 0  # Class variable
    name: str  # Instance variable
```

## Self Type (3.11+)

```python
from typing import Self

class Builder:
    def set_name(self, name: str) -> Self:
        self.name = name
        return self  # Returns Self for chaining
```

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


---

> 📘 *This lesson is part of the [Python Architecture: Patterns & Type System](https://stanza.dev/courses/python-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*