---
source_course: "python-architecture"
source_lesson: "python-architecture-protocols-runtime"
---

# Runtime Checkable Protocols

## Introduction

By default, Protocols exist only for the benefit of static type checkers -- they disappear at runtime. But sometimes you need to check whether an object satisfies a Protocol while your program is running, for example when processing a heterogeneous list of objects or dispatching to different handlers. The `@runtime_checkable` decorator makes this possible, though it comes with important limitations you need to understand.

## Key Concepts

- **@runtime_checkable**: A decorator that enables `isinstance()` and `issubclass()` checks against a Protocol at runtime.
- **Method-only checking**: At runtime, `isinstance()` only verifies that the required methods exist on the object. It does not check attribute types, return types, or method signatures.
- **Static vs runtime checking**: Static type checkers perform deep structural analysis; `@runtime_checkable` provides a shallow existence check at runtime.

## Real World Context

You are writing a resource cleanup function that iterates over a list of mixed objects. Some have a `close()` method and some do not. Instead of wrapping every call in `hasattr()` checks, you define a `Closable` runtime-checkable Protocol and use `isinstance()` to cleanly branch your logic. This is safer than `hasattr()` because it checks against a well-defined interface, not an arbitrary string.

## Deep Dive

To make a Protocol available for `isinstance()` checks, decorate it with `@runtime_checkable`. The following example defines a `Closable` Protocol and demonstrates how `isinstance()` works with it:

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

Because `File` has a `close()` method, `isinstance(obj, Closable)` returns `True`. Without the `@runtime_checkable` decorator, this call would raise a `TypeError`.

However, runtime checking has a significant limitation: it only verifies that the required methods exist. It does not check attributes, return types, or argument signatures. The following example illustrates this gap:

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

The `isinstance()` check returns `True` because `MyClass` has a `process()` method. The `data` attribute may or may not be verified depending on the Python version and the type of attribute. This shallow checking means you should not rely on `@runtime_checkable` for full Protocol conformance at runtime.

Here are examples showing where `@runtime_checkable` works well and where you should be cautious:

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

The `Callable` Protocol is a good candidate because it only requires a single dunder method. The `Complex` Protocol is riskier because the `x: int` attribute and the `str` return type of `method()` will not be verified by `isinstance()`.

## Common Pitfalls

- **Assuming full structural checks at runtime**: `isinstance()` with a runtime-checkable Protocol only checks for method existence, not signatures, return types, or attribute types. A class with `def close(self, force: bool) -> int` would still pass an `isinstance()` check against a Protocol requiring `def close(self) -> None`.
- **Forgetting the decorator**: Without `@runtime_checkable`, calling `isinstance(obj, MyProtocol)` raises a `TypeError`. If you need runtime checks, the decorator is mandatory.
- **Using runtime checks as a substitute for static analysis**: Runtime checks are a complement to static type checking, not a replacement. Lean on your type checker for full correctness and use `@runtime_checkable` only for dynamic dispatch scenarios.

## Best Practices

- **Keep runtime-checkable Protocols simple**: Stick to Protocols that declare one or two methods with no attributes. The simpler the Protocol, the more meaningful the `isinstance()` check.
- **Prefer static type checking for correctness**: Use `@runtime_checkable` for dynamic dispatch or defensive programming, but rely on mypy or pyright for full structural verification.
- **Document the limitations**: When using `@runtime_checkable` in your code, add a comment noting that the check is shallow. Future maintainers will thank you.

## Summary

- The `@runtime_checkable` decorator enables `isinstance()` checks against Protocols at runtime.
- Runtime checks only verify method existence, not signatures, return types, or attributes.
- Simple, method-only Protocols are the best candidates for `@runtime_checkable`.
- Static type checkers remain the primary tool for full Protocol conformance checking.
- Use runtime-checkable Protocols for dynamic dispatch, not as a replacement for static analysis.

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


## Resources

- [typing — runtime_checkable](https://docs.python.org/3.14/library/typing.html#typing.runtime_checkable) — Official docs for @runtime_checkable decorator

---

> 📘 *This lesson is part of the [Python Architecture: Patterns & Type System](https://stanza.dev/courses/python-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*