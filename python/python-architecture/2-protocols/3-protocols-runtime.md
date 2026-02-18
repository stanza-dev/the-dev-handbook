---
source_course: "python-architecture"
source_lesson: "python-architecture-protocols-runtime"
---

# Runtime Checking

## The @runtime_checkable Decorator

```python
from typing import Protocol, runtime_checkable

@runtime_checkable
class Closable(Protocol):
    def close(self) -> None:
        ...

class File:
    def close(self) -> None:
        print("Closed")

# Now you can use isinstance!
obj = File()
print(isinstance(obj, Closable))  # True
```

## Limitations

```python
@runtime_checkable
class HasData(Protocol):
    data: str
    def process(self) -> None: ...

class MyClass:
    data = "hello"
    def process(self) -> None:
        pass

# isinstance only checks methods exist, not attributes!
print(isinstance(MyClass(), HasData))  # True (may not check 'data')
```

## Best Practices

1. Use for simple method existence checks
2. Don't rely on for attribute checking
3. Prefer static type checking when possible

```python
# Good: Simple method protocol
@runtime_checkable
class Callable(Protocol):
    def __call__(self) -> None: ...

# Caution: Complex protocols may not check everything
@runtime_checkable
class Complex(Protocol):
    x: int
    def method(self) -> str: ...
```

## Code Examples

**Runtime protocol checking**

```python
from typing import Protocol, runtime_checkable

@runtime_checkable
class SupportsLen(Protocol):
    def __len__(self) -> int: ...

@runtime_checkable
class SupportsIter(Protocol):
    def __iter__(self): ...

def describe(obj: object) -> str:
    parts = []
    if isinstance(obj, SupportsLen):
        parts.append(f"length={len(obj)}")
    if isinstance(obj, SupportsIter):
        parts.append("iterable")
    return ", ".join(parts) or "unknown"

print(describe([1, 2, 3]))  # length=3, iterable
print(describe(42))         # unknown
```


---

> 📘 *This lesson is part of the [Python Architecture: Patterns & Type System](https://stanza.dev/courses/python-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*