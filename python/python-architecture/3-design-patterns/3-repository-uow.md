---
source_course: "python-architecture"
source_lesson: "python-architecture-repository-uow"
---

# Repository & Unit of Work Patterns

## Introduction

The **Repository** pattern abstracts data access behind a collection-like interface, hiding database details from business logic. The **Unit of Work** pattern tracks all changes made during a business transaction and commits them atomically. Together they form the backbone of clean data access layers in Python applications.

## Key Concepts

- **Repository** — mediates between the domain and the data mapping layer. It provides methods like `get`, `add`, `list`, and `delete` without exposing SQL or ORM internals.
- **Unit of Work** — maintains a list of objects affected by a business transaction and coordinates writing out changes in a single commit.

## Real World Context

| Pattern | Where You See It |
|---------|------------------|
| Repository | Django ORM managers, SQLAlchemy data access layers, FastAPI service layers |
| Unit of Work | SQLAlchemy `Session`, Django `transaction.atomic()`, any "save all or rollback" workflow |

## Deep Dive

### Repository with Protocol

Define the interface with a Protocol so the business logic never imports the ORM:

```python
from typing import Protocol
from dataclasses import dataclass

@dataclass
class User:
    id: int
    name: str
    email: str

class UserRepository(Protocol):
    def get(self, user_id: int) -> User | None: ...
    def add(self, user: User) -> None: ...
    def list_all(self) -> list[User]: ...
    def delete(self, user_id: int) -> None: ...
```

### In-Memory Implementation (for testing)

```python
class InMemoryUserRepository:
    def __init__(self) -> None:
        self._users: dict[int, User] = {}

    def get(self, user_id: int) -> User | None:
        return self._users.get(user_id)

    def add(self, user: User) -> None:
        self._users[user.id] = user

    def list_all(self) -> list[User]:
        return list(self._users.values())

    def delete(self, user_id: int) -> None:
        self._users.pop(user_id, None)
```

### SQLAlchemy Implementation

```python
from sqlalchemy.orm import Session

class SQLUserRepository:
    def __init__(self, session: Session) -> None:
        self._session = session

    def get(self, user_id: int) -> User | None:
        row = self._session.get(UserModel, user_id)
        return User(id=row.id, name=row.name, email=row.email) if row else None

    def add(self, user: User) -> None:
        self._session.add(UserModel(id=user.id, name=user.name, email=user.email))

    def list_all(self) -> list[User]:
        rows = self._session.query(UserModel).all()
        return [User(id=r.id, name=r.name, email=r.email) for r in rows]

    def delete(self, user_id: int) -> None:
        row = self._session.get(UserModel, user_id)
        if row:
            self._session.delete(row)
```

### Unit of Work

```python
from typing import Protocol

class UnitOfWork(Protocol):
    users: UserRepository

    def commit(self) -> None: ...
    def rollback(self) -> None: ...
    def __enter__(self) -> "UnitOfWork": ...
    def __exit__(self, *args) -> None: ...

class SQLUnitOfWork:
    def __init__(self, session_factory) -> None:
        self._session_factory = session_factory

    def __enter__(self) -> "SQLUnitOfWork":
        self._session = self._session_factory()
        self.users = SQLUserRepository(self._session)
        return self

    def __exit__(self, exc_type, *args) -> None:
        if exc_type:
            self.rollback()
        self._session.close()

    def commit(self) -> None:
        self._session.commit()

    def rollback(self) -> None:
        self._session.rollback()
```

### Using Repository + UoW in a Service

```python
def register_user(name: str, email: str, uow: UnitOfWork) -> User:
    with uow:
        user = User(id=next_id(), name=name, email=email)
        uow.users.add(user)
        uow.commit()
        return user
```

## Common Pitfalls

1. **Leaking ORM objects** — the Repository should return domain objects, not ORM models. Otherwise business logic becomes coupled to the database.
2. **Forgetting rollback** — always handle exceptions in the Unit of Work. The context manager pattern (`__enter__`/`__exit__`) handles this automatically.
3. **Over-abstracting** — for small scripts or CRUD apps, using the ORM directly is fine. Add Repository/UoW when you have complex business logic.

## Best Practices

- Define Repository as a **Protocol** so tests can use an in-memory implementation with zero database overhead.
- Use the **context manager** protocol (`with` statement) for Unit of Work to guarantee cleanup.
- Keep repositories **thin** — they translate between domain objects and persistence, nothing more.
- Place repositories in a dedicated `repositories/` package, separate from services and controllers.

## Summary

The Repository pattern gives your business logic a clean, collection-like API for data access, while the Unit of Work pattern ensures all changes are committed or rolled back as a single transaction. Together they keep your codebase testable, flexible, and independent of any specific database or ORM.

## Code Examples

**Repository pattern with Protocol and in-memory implementation**

```python
from typing import Protocol
from dataclasses import dataclass

@dataclass
class Product:
    id: int
    name: str
    price: float

class ProductRepository(Protocol):
    def get(self, product_id: int) -> Product | None: ...
    def add(self, product: Product) -> None: ...
    def list_all(self) -> list[Product]: ...

class InMemoryProductRepo:
    def __init__(self) -> None:
        self._store: dict[int, Product] = {}

    def get(self, product_id: int) -> Product | None:
        return self._store.get(product_id)

    def add(self, product: Product) -> None:
        self._store[product.id] = product

    def list_all(self) -> list[Product]:
        return list(self._store.values())

# Business logic depends on the Protocol, not the implementation
def most_expensive(repo: ProductRepository) -> Product | None:
    products = repo.list_all()
    return max(products, key=lambda p: p.price) if products else None

# Works with InMemoryProductRepo (tests) or SQLProductRepo (production)
repo = InMemoryProductRepo()
repo.add(Product(1, "Widget", 9.99))
repo.add(Product(2, "Gadget", 24.99))
print(most_expensive(repo))  # Product(id=2, name='Gadget', price=24.99)
```

**Unit of Work with context manager for atomic transactions**

```python
class UnitOfWork:
    """Context-manager based Unit of Work."""
    def __init__(self, session_factory) -> None:
        self._session_factory = session_factory

    def __enter__(self):
        self._session = self._session_factory()
        self.products = SQLProductRepo(self._session)
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        if exc_type:
            self._session.rollback()
        self._session.close()
        return False  # Do not suppress exceptions

    def commit(self) -> None:
        self._session.commit()

# Usage — all-or-nothing transaction
def transfer_stock(from_id: int, to_id: int, qty: int, uow: UnitOfWork):
    with uow:
        src = uow.products.get(from_id)
        dst = uow.products.get(to_id)
        src.stock -= qty
        dst.stock += qty
        uow.commit()  # Both updates or neither
```


## Resources

- [SQLAlchemy Unit of Work](https://docs.sqlalchemy.org/en/20/orm/session_basics.html) — SQLAlchemy Session as a Unit of Work implementation
- [Repository Pattern in Python](https://docs.python.org/3/library/typing.html#typing.Protocol) — Using typing.Protocol to define repository interfaces

---

> 📘 *This lesson is part of the [Python Architecture: Patterns & Type System](https://stanza.dev/courses/python-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*