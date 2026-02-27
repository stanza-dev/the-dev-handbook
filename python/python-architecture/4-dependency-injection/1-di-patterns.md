---
source_course: "python-architecture"
source_lesson: "python-architecture-di-patterns"
---

# Dependency Injection (DI)

## Introduction

Dependency Injection is a design principle where a class receives its dependencies from external sources rather than creating them internally. This pattern is fundamental to writing modular, testable, and maintainable Python applications. DI implements the broader Inversion of Control (IoC) principle: instead of a class controlling its own dependencies, control is inverted to the caller.

## Key Concepts

- **Dependency**: Any object that another object needs to function (a database connection, an API client, a logger)
- **Injection**: The act of providing a dependency to a class from outside
- **Inversion of Control (IoC)**: The principle that high-level modules should not depend on low-level modules; both should depend on abstractions
- **Composition Root**: The single place in your application where all dependencies are wired together

## Real World Context

Consider a web application with a UserService that needs a database and an email sender. Without DI, the service creates its own database connection and email client, making it impossible to test without a real database or email server. With DI, you pass these dependencies in, allowing you to substitute fakes during testing and swap implementations in production.

## Deep Dive

### Hard-Coded Dependencies (Problematic)

```python
class UserService:
    def __init__(self):
        self.db = PostgresDatabase()  # Hard dependency
        self.mailer = SmtpMailer()    # Hard dependency

    def create_user(self, name: str) -> User:
        user = User(name=name)
        self.db.save(user)
        self.mailer.send_welcome(user.email)
        return user
```

This is problematic because:
- Cannot test without a real Postgres database
- Cannot test without an SMTP server
- Cannot swap to a different database or mailer

### Injected Dependencies (Better)

```python
from typing import Protocol

class Database(Protocol):
    def save(self, entity: object) -> None: ...

class Mailer(Protocol):
    def send_welcome(self, email: str) -> None: ...

class UserService:
    def __init__(self, db: Database, mailer: Mailer):
        self.db = db
        self.mailer = mailer

    def create_user(self, name: str, email: str) -> dict:
        user = {"name": name, "email": email}
        self.db.save(user)
        self.mailer.send_welcome(email)
        return user
```

### Using `Annotated` for DI Frameworks

Python 3.9+ `Annotated` is often used by DI frameworks (like FastAPI or specialized libraries) to attach metadata to types.

```python
from typing import Annotated
from fastapi import Depends

def get_db() -> Database:
    return PostgresDatabase()

def handler(db: Annotated[Database, Depends(get_db)]):
    db.save(entity)
```

## Common Pitfalls

- **Over-injection**: Injecting too many dependencies into a single class is a sign it has too many responsibilities. If a class needs more than 4-5 dependencies, consider splitting it.
- **Service Locator anti-pattern**: Passing a container/registry and having the class look up its own dependencies defeats the purpose of DI. Dependencies should be explicit in the constructor signature.
- **Ignoring Protocols**: Injecting concrete classes instead of Protocols (interfaces) reduces flexibility. Always depend on abstractions.

## Best Practices

1. **Depend on abstractions**: Use Protocols to define what your dependencies must provide
2. **Inject through constructors**: Make dependencies required and explicit
3. **Wire at the composition root**: Configure all dependencies in one place (e.g., your main module or a factory function)
4. **Keep constructors simple**: Constructors should only assign dependencies, not perform logic

## Summary

Dependency Injection decouples classes from their dependencies by passing them in rather than creating them internally. Combined with Python's Protocol system, DI enables testable, flexible code where implementations can be swapped without modifying the dependent class. The key insight is: a class should declare what it needs, not how to create it.

## Code Examples

**Dependency injection with Protocol-based abstractions**

```python
from typing import Protocol

class Logger(Protocol):
    def log(self, message: str) -> None: ...

class ConsoleLogger:
    def log(self, message: str) -> None:
        print(f"[LOG] {message}")

class FileLogger:
    def __init__(self, path: str) -> None:
        self.path = path

    def log(self, message: str) -> None:
        with open(self.path, "a") as f:
            f.write(f"{message}\n")

class OrderService:
    def __init__(self, logger: Logger) -> None:
        self.logger = logger

    def place_order(self, item: str) -> None:
        self.logger.log(f"Order placed: {item}")

# Production: use FileLogger
service = OrderService(FileLogger("/var/log/orders.log"))
service.place_order("Widget")

# Testing: use ConsoleLogger (or a Mock)
test_service = OrderService(ConsoleLogger())
test_service.place_order("Test Widget")
```

**Using Annotated for DI metadata**

```python
from typing import Annotated

# Annotated attaches metadata to type hints
type APIKey = Annotated[str, "header-auth"]
type PositiveInt = Annotated[int, "must be > 0"]

def authenticate(key: APIKey) -> bool:
    return len(key) > 0

def set_quantity(qty: PositiveInt) -> None:
    print(f"Setting quantity to {qty}")

# Frameworks like FastAPI read the metadata
# to auto-inject dependencies at runtime
```


## Resources

- [PEP 593 - Flexible function and variable annotations](https://peps.python.org/pep-0593/) — The PEP that introduced Annotated for attaching metadata to type hints
- [Dependency Injection Principles, Practices, and Patterns](https://martinfowler.com/articles/injection.html) — Martin Fowler's foundational article on Inversion of Control and Dependency Injection

---

> 📘 *This lesson is part of the [Python Architecture: Patterns & Type System](https://stanza.dev/courses/python-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*