---
source_course: "python-architecture"
source_lesson: "python-architecture-common-patterns"
---

# Singleton, Factory & Observer Patterns

## Introduction

Design patterns are reusable solutions to common software design problems. Python's dynamic nature and first-class functions give these patterns a distinctive flavor compared to languages like Java or C++. In this lesson we explore three foundational **creational and behavioral** patterns: Singleton, Factory Method, and Observer.

## Key Concepts

- **Singleton** — ensure a class has exactly one instance and provide a global access point.
- **Factory Method** — delegate object creation to a function or subclass so callers don't depend on concrete types.
- **Observer** — define a one-to-many dependency so that when one object changes state, all dependents are notified.

## Real World Context

| Pattern | Typical Use Case |
|---------|------------------|
| Singleton | Database connection pool, application config, logger |
| Factory | Plugin loaders, notification dispatchers, serializer selection |
| Observer | Event systems, GUI callbacks, pub/sub messaging |

## Deep Dive

### Singleton via Metaclass

Python metaclasses intercept class creation, making them ideal for Singleton:

```python
class SingletonMeta(type):
    _instances: dict[type, object] = {}

    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]

class AppConfig(metaclass=SingletonMeta):
    def __init__(self):
        self.debug = False

a = AppConfig()
b = AppConfig()
assert a is b  # True — same instance
```

### Factory Method

Using a simple factory function with a registry:

```python
from typing import Protocol

class Notifier(Protocol):
    def send(self, message: str) -> None: ...

class EmailNotifier:
    def send(self, message: str) -> None:
        print(f"Email: {message}")

class SlackNotifier:
    def send(self, message: str) -> None:
        print(f"Slack: {message}")

_registry: dict[str, type[Notifier]] = {
    "email": EmailNotifier,
    "slack": SlackNotifier,
}

def create_notifier(channel: str) -> Notifier:
    cls = _registry.get(channel)
    if cls is None:
        raise ValueError(f"Unknown channel: {channel}")
    return cls()
```

### Observer

A lightweight event system using callbacks:

```python
from typing import Callable

class EventEmitter:
    def __init__(self) -> None:
        self._listeners: dict[str, list[Callable]] = {}

    def on(self, event: str, callback: Callable) -> None:
        self._listeners.setdefault(event, []).append(callback)

    def emit(self, event: str, *args) -> None:
        for cb in self._listeners.get(event, []):
            cb(*args)

emitter = EventEmitter()
emitter.on("user_created", lambda name: print(f"Welcome {name}"))
emitter.emit("user_created", "Alice")
```

## Common Pitfalls

1. **Singleton and testing** — global state makes unit tests order-dependent. Prefer dependency injection and use Singleton sparingly.
2. **Factory string keys** — using raw strings for the registry is fragile. Consider `Literal` types or an `Enum` for the keys.
3. **Observer memory leaks** — holding strong references to callbacks can prevent garbage collection. Use `weakref` for long-lived emitters.

## Best Practices

- Favor **module-level instances** over metaclass Singletons when a simple global is sufficient.
- Combine Factory with **Protocol** so the return type is an interface, not a concrete class.
- Keep Observer callbacks **small and side-effect-free**; delegate heavy work to a service layer.

## Summary

Singleton, Factory, and Observer solve object creation and communication problems that appear in almost every codebase. Python's metaclasses, first-class functions, and Protocols let you implement them with less boilerplate than in statically-typed languages. Use Singleton for truly shared resources, Factory to decouple creation from usage, and Observer to keep components loosely coupled.

## Code Examples

**Singleton pattern using a metaclass**

```python
class SingletonMeta(type):
    """Metaclass that ensures only one instance per class."""
    _instances: dict[type, object] = {}

    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]

class AppConfig(metaclass=SingletonMeta):
    def __init__(self) -> None:
        self.debug = False
        self.version = "1.0.0"

# Both variables reference the exact same object
config_a = AppConfig()
config_b = AppConfig()
assert config_a is config_b
print(config_a.version)  # 1.0.0
```

**Factory Method with Protocol return type**

```python
from typing import Protocol, Callable

# --- Factory Method ---
class Serializer(Protocol):
    def serialize(self, data: dict) -> str: ...

class JSONSerializer:
    def serialize(self, data: dict) -> str:
        import json
        return json.dumps(data)

class XMLSerializer:
    def serialize(self, data: dict) -> str:
        items = "".join(f"<{k}>{v}</{k}>" for k, v in data.items())
        return f"<root>{items}</root>"

def get_serializer(fmt: str) -> Serializer:
    mapping: dict[str, type[Serializer]] = {
        "json": JSONSerializer,
        "xml": XMLSerializer,
    }
    return mapping[fmt]()

serializer = get_serializer("json")
print(serializer.serialize({"name": "Alice"}))
```

**Observer pattern with EventEmitter**

```python
from typing import Callable

class EventEmitter:
    """Simple Observer / pub-sub implementation."""
    def __init__(self) -> None:
        self._listeners: dict[str, list[Callable]] = {}

    def on(self, event: str, callback: Callable) -> None:
        self._listeners.setdefault(event, []).append(callback)

    def off(self, event: str, callback: Callable) -> None:
        self._listeners.get(event, []).remove(callback)

    def emit(self, event: str, *args, **kwargs) -> None:
        for cb in self._listeners.get(event, []):
            cb(*args, **kwargs)

# Usage
bus = EventEmitter()
bus.on("order_placed", lambda order_id: print(f"Processing {order_id}"))
bus.on("order_placed", lambda order_id: print(f"Emailing receipt for {order_id}"))
bus.emit("order_placed", "ORD-42")
```


## Resources

- [Python Design Patterns Guide](https://docs.python.org/3/faq/programming.html#how-do-i-share-global-variables-across-modules) — Python FAQ on sharing global state across modules
- [PEP 3119 — Abstract Base Classes](https://peps.python.org/pep-3119/) — Foundation for interface-based patterns in Python

---

> 📘 *This lesson is part of the [Python Architecture: Patterns & Type System](https://stanza.dev/courses/python-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*