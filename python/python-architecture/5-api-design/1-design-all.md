---
source_course: "python-architecture"
source_lesson: "python-architecture-api-design-all"
---

# Public Interface Design

## The `__all__` Variable
By default, `from module import *` imports everything that doesn't start with an underscore. Defining `__all__` explicitly controls what is exported.

```python
__all__ = ['PublicClass', 'public_func']

class PublicClass: ...
class _PrivateClass: ...
```

## ABCs (Abstract Base Classes)
Use `abc.ABC` and `@abstractmethod` to enforce interface contracts.

## Code Examples

**Abstract Base Class**

```python
from abc import ABC, abstractmethod

class Stream(ABC):
    @abstractmethod
    def read(self):
        pass

# class FileStream(Stream):
#    pass 
# This raises TypeError because read() is not implemented
```


---

> 📘 *This lesson is part of the [Python Architecture: Patterns & Type System](https://stanza.dev/courses/python-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*