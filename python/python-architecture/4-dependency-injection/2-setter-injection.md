---
source_course: "python-architecture"
source_lesson: "python-architecture-constructor-vs-setter-injection"
---

# Constructor vs Setter Injection

## Introduction

There are multiple ways to inject dependencies into a class. The two most common approaches are constructor injection (passing dependencies via `__init__`) and setter injection (assigning dependencies via methods or properties after construction). A third, less common style is method injection, where a dependency is passed directly to the method that needs it. Understanding the trade-offs helps you choose the right approach for each situation.

## Key Concepts

- **Constructor Injection**: Dependencies are passed as arguments to `__init__`. The object is fully initialized and ready to use immediately after construction.
- **Setter Injection**: Dependencies are assigned after construction via setter methods or property assignment. The object may be in an incomplete state until all setters are called.
- **Method Injection**: A dependency is passed as a parameter to a specific method, used when the dependency varies per call.
- **Immutability**: Constructor injection naturally supports immutable objects; setter injection does not.

## Real World Context

In frameworks like Django, some dependencies (such as the request object) are naturally provided via method injection because they change per request. However, core services like the database connection are best provided via constructor injection because they remain constant for the lifetime of the service. Setter injection is sometimes necessary for circular dependencies or optional features, but it should be the exception, not the rule.

## Deep Dive

### Constructor Injection (Preferred)

```python
from typing import Protocol

class Cache(Protocol):
    def get(self, key: str) -> str | None: ...
    def set(self, key: str, value: str) -> None: ...

class ProductService:
    """All dependencies are required and provided at construction time."""
    def __init__(self, db: Database, cache: Cache) -> None:
        self._db = db
        self._cache = cache

    def get_product(self, product_id: str) -> dict:
        cached = self._cache.get(product_id)
        if cached:
            return {"id": product_id, "name": cached}
        product = self._db.query(product_id)
        self._cache.set(product_id, product["name"])
        return product
```

Advantages:
- Object is always in a valid state
- Dependencies are explicit and documented
- Supports immutability (use `@dataclass(frozen=True)`)
- Easy to see when a class has too many dependencies

### Setter Injection

```python
class NotificationService:
    """Logger is optional and can be set after construction."""
    def __init__(self, mailer: Mailer) -> None:
        self._mailer = mailer
        self._logger: Logger | None = None

    def set_logger(self, logger: Logger) -> None:
        self._logger = logger

    def notify(self, email: str, message: str) -> None:
        if self._logger:
            self._logger.log(f"Sending to {email}")
        self._mailer.send(email, message)
```

Advantages:
- Useful for optional dependencies
- Can break circular dependency chains
- Allows reconfiguration after construction

Disadvantages:
- Object may be used before being fully configured
- Harder to reason about object state
- No compile-time guarantee all dependencies are set

### Method Injection

```python
class ReportGenerator:
    def __init__(self, db: Database) -> None:
        self._db = db

    def generate(self, formatter: Formatter) -> str:
        """Formatter varies per call, so it is method-injected."""
        data = self._db.fetch_report_data()
        return formatter.format(data)
```

Use method injection when the dependency is specific to a single operation and may change between calls.

## Common Pitfalls

- **Using setter injection for required dependencies**: If the object cannot function without a dependency, it must be constructor-injected. Setter injection for required dependencies leads to runtime NoneType errors.
- **Temporal coupling**: With setter injection, the order in which setters are called matters. Forgetting one setter or calling them out of order leads to subtle bugs.
- **Mutable state confusion**: Allowing dependencies to be swapped at runtime via setters makes it harder to reason about the object's behavior at any given point in time.

## Best Practices

1. **Default to constructor injection** for all required dependencies
2. **Use setter injection only for truly optional** dependencies (e.g., an optional logger)
3. **Use method injection** when the dependency varies per invocation
4. **Use `@dataclass` for clean constructor injection**:

```python
from dataclasses import dataclass

@dataclass
class UserService:
    db: Database
    cache: Cache
    mailer: Mailer

    def create_user(self, name: str) -> dict:
        user = {"name": name}
        self.db.save(user)
        return user
```

5. **Guard against None** if using setter injection:

```python
def notify(self, email: str) -> None:
    if self._logger is None:
        raise RuntimeError("Logger not configured")
    self._logger.log(f"Notifying {email}")
```

## Summary

Constructor injection is the preferred style in Python because it ensures objects are always fully initialized, makes dependencies explicit, and supports immutability. Setter injection is reserved for optional dependencies. Method injection is ideal when a dependency changes per call. The guiding principle: required dependencies go in the constructor, optional ones use setters, and per-call dependencies are method parameters.

## Code Examples

**Constructor injection with dataclass and test doubles**

```python
from dataclasses import dataclass
from typing import Protocol

class Repository(Protocol):
    def find(self, id: str) -> dict | None: ...
    def save(self, entity: dict) -> None: ...

class EventBus(Protocol):
    def publish(self, event: str, data: dict) -> None: ...

# Constructor injection with @dataclass
@dataclass
class AccountService:
    repo: Repository
    events: EventBus

    def close_account(self, account_id: str) -> bool:
        account = self.repo.find(account_id)
        if account is None:
            return False
        account["status"] = "closed"
        self.repo.save(account)
        self.events.publish("account.closed", account)
        return True

# Easy to test: pass fakes directly
class FakeRepo:
    def __init__(self) -> None:
        self.store: dict[str, dict] = {}
    def find(self, id: str) -> dict | None:
        return self.store.get(id)
    def save(self, entity: dict) -> None:
        self.store[entity["id"]] = entity

class FakeEventBus:
    def __init__(self) -> None:
        self.events: list[tuple[str, dict]] = []
    def publish(self, event: str, data: dict) -> None:
        self.events.append((event, data))

repo = FakeRepo()
repo.store["acc-1"] = {"id": "acc-1", "status": "active"}
bus = FakeEventBus()

service = AccountService(repo=repo, events=bus)
assert service.close_account("acc-1") is True
assert bus.events[0][0] == "account.closed"
```

**Method injection for per-call dependencies**

```python
from typing import Protocol

class Formatter(Protocol):
    def format(self, data: list[dict]) -> str: ...

class CSVFormatter:
    def format(self, data: list[dict]) -> str:
        if not data:
            return ""
        headers = ",".join(data[0].keys())
        rows = "\n".join(",".join(str(v) for v in row.values()) for row in data)
        return f"{headers}\n{rows}"

class JSONFormatter:
    def format(self, data: list[dict]) -> str:
        import json
        return json.dumps(data, indent=2)

class ReportService:
    def __init__(self, db) -> None:
        self._db = db  # Constructor-injected (stable)

    def export(self, formatter: Formatter) -> str:
        """Formatter is method-injected (varies per call)."""
        data = self._db.fetch_all()
        return formatter.format(data)

# Different formatters for different calls
# report.export(CSVFormatter())
# report.export(JSONFormatter())
```


## Resources

- [Python dataclasses documentation](https://docs.python.org/3/library/dataclasses.html) — Official docs for @dataclass, which simplifies constructor injection patterns
- [Inversion of Control Containers and the Dependency Injection Pattern](https://martinfowler.com/articles/injection.html) — Martin Fowler's detailed comparison of constructor, setter, and interface injection

---

> 📘 *This lesson is part of the [Python Architecture: Patterns & Type System](https://stanza.dev/courses/python-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*