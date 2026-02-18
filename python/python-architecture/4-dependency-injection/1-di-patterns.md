---
source_course: "python-architecture"
source_lesson: "python-architecture-di-patterns"
---

# Dependency Injection (DI)

DI makes code modular and testable. Instead of a class creating its dependencies, they are passed to it.

## Hard-Coded (Bad)
```python
class Service:
    def __init__(self):
        self.db = Database() # Hard dependency
```

## Injected (Good)
```python
class Service:
    def __init__(self, db: DatabaseProtocol):
        self.db = db
```

## Using `Annotated` for DI Frameworks
Python 3.9+ `Annotated` is often used by DI frameworks (like FastAPI or specialized libraries) to attach metadata to types.

```python
def handler(db: Annotated[Session, Depends(get_db)]):
    ...
```

## Code Examples

**Using Annotated**

```python
from typing import Annotated

# Metadata can be anything
type APIKey = Annotated[str, "header-auth"]

def authenticate(key: APIKey):
    print(f"Validating {key}")
```


---

> 📘 *This lesson is part of the [Python Architecture: Patterns & Type System](https://stanza.dev/courses/python-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*