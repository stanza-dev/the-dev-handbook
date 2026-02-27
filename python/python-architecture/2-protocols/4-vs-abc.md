---
source_course: "python-architecture"
source_lesson: "python-architecture-protocols-vs-abc"
---

# Protocols vs Abstract Base Classes

## Introduction

Python gives you two ways to define interfaces: Abstract Base Classes (ABCs) and Protocols. They solve the same fundamental problem -- specifying what methods a class must implement -- but they use opposite strategies. Understanding when to reach for each one is a key architectural decision that affects how tightly coupled your code becomes.

## Key Concepts

- **Nominal typing (ABCs)**: A class satisfies an interface only if it explicitly inherits from it. The relationship is declared in the class definition.
- **Structural typing (Protocols)**: A class satisfies an interface if it has the right methods, regardless of its inheritance hierarchy. The relationship is inferred by the type checker.
- **Interface segregation**: The principle that clients should not be forced to depend on interfaces they do not use. Both ABCs and Protocols can enforce this, but Protocols make it easier.

## Real World Context

You are building a web framework with an internal handler pipeline and a public plugin API. Internally, you want every handler to inherit shared logging and error-handling logic, so an ABC with concrete methods makes sense. But for external plugin authors, you cannot force them to inherit from your base class -- they might already have their own class hierarchy. A Protocol lets you define the contract ("must have a `handle()` method") without imposing any inheritance requirement.

## Deep Dive

Abstract Base Classes use **nominal typing**: a subclass must explicitly inherit from the ABC, and the ABC can enforce that abstract methods are implemented. Here is a classic ABC example:

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

If `Dog` does not inherit from `Animal`, it will not be recognized as an `Animal` by the type checker, even if it has a `speak()` method. The inheritance is mandatory. This is useful when you want to enforce a strict hierarchy and catch missing implementations at instantiation time (Python raises `TypeError` if you try to instantiate a class that has not implemented all abstract methods).

Protocols use **structural typing**: a class satisfies the Protocol if it has the right methods, no inheritance needed. The same scenario with a Protocol looks like this:

```python
from typing import Protocol

class Speaker(Protocol):
    def speak(self) -> str:
        ...

class Dog:  # No inheritance needed
    def speak(self) -> str:
        return "Woof"
```

`Dog` satisfies `Speaker` simply because it has a `speak() -> str` method. There is no coupling between the two classes. This is ideal when you are defining contracts for code you do not control.

The following table summarizes when to choose each approach:

| Use Case | Choose |
|----------|--------|
| Control implementation | ABC |
| Third-party code | Protocol |
| Duck typing with types | Protocol |
| Shared functionality | ABC |
| Simple interfaces | Protocol |

ABCs excel when you need shared concrete methods (like logging or validation) that all subclasses inherit. Protocols excel when you need lightweight contracts across module or package boundaries.

In practice, you can combine both in the same codebase. Use an ABC for your internal implementation hierarchy where you want shared behavior, and expose a Protocol as the public-facing interface. The following example shows this pattern:

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

Your internal handlers inherit from `BaseHandler` and get the `log()` method for free. External plugins only need to satisfy the `Handler` Protocol by implementing `handle()`. Both internal and external handlers can be passed to any function that accepts `Handler`, because `BaseHandler` subclasses structurally satisfy the Protocol.

## Common Pitfalls

- **Using ABCs for everything**: If your ABC has no concrete methods and no shared state, it is just a Protocol with extra coupling. Switch to a Protocol and remove the inheritance requirement.
- **Mixing up instantiation errors with type errors**: ABCs raise `TypeError` at runtime if abstract methods are missing. Protocols catch missing methods at static analysis time. These are different feedback loops -- do not assume one covers the other.
- **Forgetting that ABCs provide runtime enforcement**: Unlike Protocols, ABCs prevent you from instantiating a class with missing abstract methods. If you need this runtime safety net (e.g., in a framework where plugin authors might skip type checking), ABCs are the better choice.

## Best Practices

- **Use Protocols for boundary contracts**: At the edges of your system (APIs, plugins, adapters), Protocols keep coupling low and flexibility high.
- **Use ABCs for shared implementation**: When subclasses need to inherit concrete methods, an ABC provides both the contract and the shared code in one place.
- **Combine both strategically**: Define an ABC for your internal hierarchy and a Protocol for the public interface. This gives you the best of both worlds -- shared implementation internally and loose coupling externally.

## Summary

- ABCs use nominal typing (explicit inheritance), Protocols use structural typing (method matching).
- ABCs are best when you need shared concrete methods or runtime enforcement of abstract method implementation.
- Protocols are best for lightweight contracts, third-party code integration, and boundary interfaces.
- You can combine both in the same codebase: ABCs for internal hierarchies, Protocols for external contracts.
- Choose based on whether you need shared behavior (ABC) or loose coupling (Protocol).

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


## Resources

- [abc — Abstract Base Classes](https://docs.python.org/3.14/library/abc.html) — Official ABC module documentation
- [typing — Protocol](https://docs.python.org/3.14/library/typing.html#typing.Protocol) — Official Protocol documentation for comparison

---

> 📘 *This lesson is part of the [Python Architecture: Patterns & Type System](https://stanza.dev/courses/python-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*