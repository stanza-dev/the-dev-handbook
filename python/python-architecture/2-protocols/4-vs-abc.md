---
source_course: "python-architecture"
source_lesson: "python-architecture-protocols-vs-abc"
---

# Protocols vs ABCs

## Abstract Base Classes (Nominal Typing)

```python
from abc import ABC, abstractmethod

class Animal(ABC):
    @abstractmethod
    def speak(self) -> str:
        pass

class Dog(Animal):  # Must inherit!
    def speak(self) -> str:
        return "Woof"
```

## Protocols (Structural Typing)

```python
from typing import Protocol

class Speaker(Protocol):
    def speak(self) -> str:
        ...

class Dog:  # No inheritance needed
    def speak(self) -> str:
        return "Woof"
```

## When to Use Each

| Use Case | Choose |
|----------|--------|
| Control implementation | ABC |
| Third-party code | Protocol |
| Duck typing with types | Protocol |
| Shared functionality | ABC |
| Simple interfaces | Protocol |

## Combining Both

```python
from abc import ABC, abstractmethod
from typing import Protocol

# ABC for internal implementation
class BaseHandler(ABC):
    @abstractmethod
    def handle(self, data: str) -> None:
        pass
    
    def log(self, msg: str) -> None:
        print(f"[LOG] {msg}")

# Protocol for external interface
class Handler(Protocol):
    def handle(self, data: str) -> None:
        ...
```

## Code Examples

**Protocol for external code**

```python
from typing import Protocol

# Protocol for external libraries
class JSONSerializer(Protocol):
    def to_json(self) -> str: ...

# Works with any class that has to_json()
import json

class User:
    def __init__(self, name: str):
        self.name = name
    
    def to_json(self) -> str:
        return json.dumps({"name": self.name})

def save(obj: JSONSerializer) -> None:
    print(obj.to_json())

save(User("Alice"))  # Works without inheritance!
```


---

> 📘 *This lesson is part of the [Python Architecture: Patterns & Type System](https://stanza.dev/courses/python-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*