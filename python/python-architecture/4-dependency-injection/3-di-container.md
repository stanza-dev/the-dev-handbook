---
source_course: "python-architecture"
source_lesson: "python-architecture-building-di-container"
---

# Building a Simple DI Container

## Introduction

A DI container (also called an IoC container) is an object that knows how to create and wire dependencies together. While Python does not require a container for DI (you can wire everything manually), a lightweight container reduces boilerplate in larger applications. In this lesson, we build a simple container from scratch using Python's typing system and Protocols, demonstrating the mechanics behind frameworks like `dependency-injector` and FastAPI's `Depends`.

## Key Concepts

- **Container**: A registry that maps abstract types (Protocols) to concrete implementations or factory functions
- **Registration**: Telling the container which concrete class to use for a given abstraction
- **Resolution**: Asking the container to create an instance, automatically resolving its dependencies
- **Singleton scope**: A dependency that is created once and reused for all subsequent requests
- **Transient scope**: A new instance is created every time the dependency is requested

## Real World Context

In a real application, you might have dozens of services that depend on each other. Manually wiring `UserService(db=PostgresDB(), cache=RedisCache(), mailer=SmtpMailer())` everywhere is tedious and error-prone. A container centralizes this wiring: you register each Protocol with its implementation once, then resolve any service and the container builds the entire dependency tree automatically.

## Deep Dive

### A Minimal Container

```python
from typing import Any, TypeVar, get_type_hints

T = TypeVar("T")

class Container:
    """A simple DI container that resolves constructor dependencies."""

    def __init__(self) -> None:
        self._registry: dict[type, type] = {}
        self._singletons: dict[type, Any] = {}

    def register(self, abstract: type, concrete: type) -> None:
        """Register a concrete implementation for an abstract type."""
        self._registry[abstract] = concrete

    def register_singleton(self, abstract: type, concrete: type) -> None:
        """Register as singleton (created once, reused)."""
        self._registry[abstract] = concrete
        self._singletons[abstract] = None  # Marker for singleton

    def resolve(self, abstract: type[T]) -> T:
        """Resolve an abstract type to a concrete instance."""
        # Check singleton cache
        if abstract in self._singletons and self._singletons[abstract] is not None:
            return self._singletons[abstract]

        concrete = self._registry.get(abstract, abstract)
        hints = get_type_hints(concrete.__init__)
        hints.pop("return", None)

        # Recursively resolve constructor dependencies
        deps = {}
        for name, dep_type in hints.items():
            deps[name] = self.resolve(dep_type)

        instance = concrete(**deps)

        # Cache if singleton
        if abstract in self._singletons:
            self._singletons[abstract] = instance

        return instance
```

### Using the Container

```python
from typing import Protocol

class Database(Protocol):
    def query(self, sql: str) -> list[dict]: ...

class Cache(Protocol):
    def get(self, key: str) -> str | None: ...

class PostgresDB:
    def query(self, sql: str) -> list[dict]:
        return [{"id": 1}]

class RedisCache:
    def get(self, key: str) -> str | None:
        return None

class UserService:
    def __init__(self, db: Database, cache: Cache) -> None:
        self.db = db
        self.cache = cache

# Wire everything up
container = Container()
container.register_singleton(Database, PostgresDB)
container.register(Cache, RedisCache)

# Resolve with automatic dependency injection
user_service = container.resolve(UserService)
# Container creates PostgresDB, RedisCache, then UserService
```

### Adding Factory Functions

For more control, support factory callables:

```python
from typing import Callable

class Container:
    def __init__(self) -> None:
        self._factories: dict[type, Callable] = {}

    def register_factory(self, abstract: type, factory: Callable[[], Any]) -> None:
        self._factories[abstract] = factory

    def resolve(self, abstract: type[T]) -> T:
        if abstract in self._factories:
            return self._factories[abstract]()
        # ... fall back to auto-resolution

# Usage
container.register_factory(Database, lambda: PostgresDB(host="localhost", port=5432))
```

## Common Pitfalls

- **Circular dependencies**: If ServiceA depends on ServiceB and ServiceB depends on ServiceA, the container will recurse infinitely. Detect cycles by tracking the resolution stack.
- **Hidden dependencies**: A container can make it too easy to add dependencies. Keep an eye on the number of constructor parameters.
- **Over-engineering**: For small applications with fewer than 10 services, manual wiring at the composition root is simpler and more explicit than a container.
- **Deferred annotations**: In Python 3.14, annotations are evaluated lazily. `typing.get_type_hints()` and `annotationlib.get_annotations()` handle this correctly, but accessing `__annotations__` directly may trigger unexpected evaluation. Always use `get_type_hints()` in your container.

## Best Practices

1. **Start without a container**: Wire dependencies manually first. Only introduce a container when manual wiring becomes painful (typically 15+ services).
2. **Keep the container at the composition root**: Never pass the container into your services (that is the Service Locator anti-pattern).
3. **Prefer constructor injection**: The container should resolve constructor parameters, not set attributes after construction.
4. **Add cycle detection**: Track which types are currently being resolved and raise an error if a cycle is detected.
5. **Test the container itself**: Write tests that verify your registrations produce the expected concrete types.

```python
def test_container_resolves_user_service():
    container = Container()
    container.register(Database, PostgresDB)
    container.register(Cache, RedisCache)
    service = container.resolve(UserService)
    assert isinstance(service.db, PostgresDB)
    assert isinstance(service.cache, RedisCache)
```

## Summary

A DI container automates the wiring of dependencies by inspecting constructor type hints and recursively resolving each dependency. Python's `get_type_hints()` makes this possible without decorators or configuration files. While containers reduce boilerplate in large applications, they add complexity, so start with manual wiring and graduate to a container only when needed. The key principle: the container lives at the edge of your application and never leaks into your business logic.

## Code Examples

**Complete DI container with cycle detection and singleton support**

```python
from typing import Any, TypeVar, get_type_hints, Protocol

T = TypeVar("T")

class Container:
    def __init__(self) -> None:
        self._registry: dict[type, type] = {}
        self._singletons: dict[type, Any] = {}
        self._resolving: set[type] = set()  # Cycle detection

    def register(self, abstract: type, concrete: type) -> None:
        self._registry[abstract] = concrete

    def register_singleton(self, abstract: type, concrete: type) -> None:
        self._registry[abstract] = concrete
        self._singletons[abstract] = None

    def resolve(self, abstract: type[T]) -> T:
        if abstract in self._singletons and self._singletons[abstract] is not None:
            return self._singletons[abstract]

        if abstract in self._resolving:
            raise RuntimeError(f"Circular dependency detected for {abstract}")
        self._resolving.add(abstract)

        try:
            concrete = self._registry.get(abstract, abstract)
            hints = get_type_hints(concrete.__init__)
            hints.pop("return", None)
            deps = {name: self.resolve(t) for name, t in hints.items()}
            instance = concrete(**deps)
        finally:
            self._resolving.discard(abstract)

        if abstract in self._singletons:
            self._singletons[abstract] = instance
        return instance

# --- Usage ---
class Logger(Protocol):
    def log(self, msg: str) -> None: ...

class Database(Protocol):
    def query(self, sql: str) -> list: ...

class ConsoleLogger:
    def log(self, msg: str) -> None:
        print(msg)

class SQLiteDB:
    def __init__(self, logger: Logger) -> None:
        self.logger = logger
    def query(self, sql: str) -> list:
        self.logger.log(f"Executing: {sql}")
        return []

class UserService:
    def __init__(self, db: Database, logger: Logger) -> None:
        self.db = db
        self.logger = logger
    def list_users(self) -> list:
        self.logger.log("Fetching users")
        return self.db.query("SELECT * FROM users")

# Wire
c = Container()
c.register_singleton(Logger, ConsoleLogger)
c.register(Database, SQLiteDB)
svc = c.resolve(UserService)
svc.list_users()
# Output: Fetching users
#         Executing: SELECT * FROM users
```

**Decorator-based container registration pattern**

```python
from typing import Protocol

# Decorator-based registration
class Container:
    _instance: 'Container | None' = None
    _registry: dict[type, type] = {}

    @classmethod
    def get(cls) -> 'Container':
        if cls._instance is None:
            cls._instance = cls()
        return cls._instance

def provides(abstract: type):
    """Decorator to register a class as the provider for an abstract type."""
    def decorator(concrete: type) -> type:
        Container._registry[abstract] = concrete
        return concrete
    return decorator

# --- Usage with decorators ---
class Notifier(Protocol):
    def send(self, to: str, msg: str) -> None: ...

@provides(Notifier)
class EmailNotifier:
    def send(self, to: str, msg: str) -> None:
        print(f"Email to {to}: {msg}")

# Registration happens at import time via the decorator
print(Container._registry)  # {Notifier: EmailNotifier}
```


## Resources

- [get_type_hints documentation](https://docs.python.org/3/library/typing.html#typing.get_type_hints) — Python docs for get_type_hints(), the function that powers runtime type introspection for DI containers
- [dependency-injector library](https://python-dependency-injector.ets-labs.org/) — A mature Python DI framework that implements containers, providers, and wiring
- [The Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html) — Uncle Bob's Clean Architecture article explaining how DI fits into larger architectural patterns

---

> 📘 *This lesson is part of the [Python Architecture: Patterns & Type System](https://stanza.dev/courses/python-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*