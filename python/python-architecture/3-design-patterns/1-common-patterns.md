---
source_course: "python-architecture"
source_lesson: "python-architecture-common-patterns"
---

# Design Patterns

## Singleton
Ensuring a class has only one instance.

## Factory Method
Delegating object creation to subclasses or a separate method, useful for decoupling code from specific classes.

## Observer
A subscription mechanism to notify multiple objects about events.

## Code Examples

**Singleton Metaclass**

```python
class Singleton(type):
    _instances = {}
    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]

class DB(metaclass=Singleton):
    pass
```


---

> 📘 *This lesson is part of the [Python Architecture: Patterns & Type System](https://stanza.dev/courses/python-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*