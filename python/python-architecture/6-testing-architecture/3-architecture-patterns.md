---
source_course: "python-architecture"
source_lesson: "python-architecture-test-architecture-patterns"
---

# Test Architecture Patterns

## Introduction

Writing individual tests is straightforward, but organizing a test suite that remains maintainable as a codebase grows requires deliberate architectural decisions. This lesson covers the foundational patterns for structuring tests: the Arrange/Act/Assert pattern, test doubles taxonomy, fixtures for shared setup, and strategies for test isolation.

## Key Concepts

- **Arrange/Act/Assert (AAA)**: The three-phase structure that every well-written test follows.
- **Test Doubles Taxonomy**: The precise vocabulary for different kinds of stand-in objects — stubs, mocks, spies, and fakes.
- **Fixtures**: Reusable setup and teardown logic shared across tests.
- **Test Isolation**: Ensuring each test is independent and produces the same result regardless of execution order.

## Real World Context

Large Python projects like Django (over 10,000 tests), CPython, and SQLAlchemy rely on clear test architecture to keep their suites fast and maintainable. The pytest framework, used by the vast majority of Python projects, was designed around these patterns — its fixture system, in particular, directly supports the test isolation and setup-sharing patterns discussed here.

## Deep Dive

### Arrange / Act / Assert (AAA)

Every test should have three clearly separated phases:

```python
def test_user_receives_welcome_email():
    # Arrange: set up preconditions
    user = User(name='Alice', email='alice@test.com')
    mailer = FakeMailer()
    service = SignupService(mailer=mailer)

    # Act: perform the action under test
    service.register(user)

    # Assert: verify the expected outcome
    assert len(mailer.sent) == 1
    assert mailer.sent[0].to == 'alice@test.com'
    assert 'Welcome' in mailer.sent[0].subject
```

Keep each phase focused. If your Arrange section is 30 lines long, extract a fixture or helper function. If you have multiple Act steps, you are probably testing two things — split into two tests.

### Test Doubles Taxonomy

The term "mock" is often used loosely, but there are four distinct types of test doubles:

| Double | Purpose | Behavior |
|--------|---------|----------|
| **Stub** | Provide canned answers | Returns pre-configured values, no verification |
| **Mock** | Verify interactions | Records calls, asserts they happened correctly |
| **Spy** | Observe without replacing | Wraps the real object, records calls but delegates to real implementation |
| **Fake** | Lightweight implementation | A working but simplified version (e.g., in-memory database) |

```python
from unittest.mock import Mock, patch, call

# STUB: just returns a value, no assertions
stub_repo = Mock()
stub_repo.find_by_id.return_value = {'id': 1, 'name': 'Alice'}

# MOCK: we assert how it was called
mock_notifier = Mock()
service.process(user)
mock_notifier.send.assert_called_once_with(user_id=1, message='Done')

# SPY: wraps the real object
real_cache = RedisCache()
with patch.object(real_cache, 'set', wraps=real_cache.set) as spy_set:
    service.update(key='x', value=42)
    spy_set.assert_called_once()  # Verified, but real set() ran too

# FAKE: simplified working implementation
class FakeUserRepository:
    def __init__(self):
        self._users = {}
        self._next_id = 1

    def save(self, user: dict) -> int:
        uid = self._next_id
        self._users[uid] = {**user, 'id': uid}
        self._next_id += 1
        return uid

    def find_by_id(self, uid: int) -> dict | None:
        return self._users.get(uid)
```

### Pytest Fixtures

Fixtures in pytest provide reusable setup logic with automatic teardown:

```python
import pytest
from unittest.mock import Mock

@pytest.fixture
def mock_db():
    """Provide a mock database for each test."""
    db = Mock()
    db.is_connected.return_value = True
    return db

@pytest.fixture
def user_service(mock_db):
    """Provide a UserService with injected mock DB."""
    return UserService(db=mock_db)

def test_create_user(user_service, mock_db):
    mock_db.insert.return_value = {'id': 1, 'name': 'Alice'}
    user = user_service.create('Alice')
    assert user['name'] == 'Alice'
    mock_db.insert.assert_called_once()

def test_delete_user(user_service, mock_db):
    mock_db.delete.return_value = True
    assert user_service.delete(1) is True
```

### Fixture Scopes

```python
import pytest

@pytest.fixture(scope='function')  # Default: fresh per test
def fresh_db(): ...

@pytest.fixture(scope='class')  # Shared within a test class
def class_db(): ...

@pytest.fixture(scope='module')  # Shared within a module
def module_db(): ...

@pytest.fixture(scope='session')  # Shared across entire test run
def session_db(): ...
```

### Test Isolation Strategies

Tests must not depend on each other. Key strategies:

1. **Fresh fixtures per test**: Use `scope='function'` (the default) so each test gets clean state.
2. **Database transactions**: Wrap each test in a transaction and roll back afterward.
3. **Temporary directories**: Use `tmp_path` fixture for file I/O tests.
4. **Environment variable isolation**: Use `monkeypatch` to set/unset env vars without affecting other tests.

```python
def test_config_reads_env(monkeypatch):
    monkeypatch.setenv('API_KEY', 'test-key-123')
    config = load_config()
    assert config.api_key == 'test-key-123'

def test_file_processing(tmp_path):
    test_file = tmp_path / 'data.csv'
    test_file.write_text('name,age\nAlice,30')
    result = process_csv(test_file)
    assert result[0]['name'] == 'Alice'
```

## Common Pitfalls

1. **Shared mutable state between tests**: Using module-level variables or class attributes that one test modifies and another reads. Use fixtures with function scope instead.
2. **Order-dependent tests**: Tests that pass only when run in a specific order. Use `pytest-randomly` to detect this.
3. **Confusing stubs and mocks**: Using assertion methods on objects that are only meant to provide canned data. Be intentional about which type of double you need.
4. **Fixture dependency chains that are too deep**: If a fixture depends on five other fixtures, the setup becomes hard to understand. Keep chains shallow (2-3 levels max).

## Best Practices

- Name tests descriptively: `test_user_with_expired_token_gets_401`, not `test_user_3`.
- One logical assertion per test (multiple `assert` lines are fine if they verify one behavior).
- Use the AAA pattern consistently — add blank lines between the three sections for readability.
- Prefer fakes over mocks for complex dependencies (databases, caches) to catch more integration issues.
- Run tests with `pytest --strict-markers -x -v` during development for immediate feedback.

## Summary

Well-structured tests follow the Arrange/Act/Assert pattern, use the right type of test double (stub, mock, spy, or fake), share setup through fixtures, and maintain strict isolation between tests. These patterns keep test suites fast, readable, and reliable as the codebase grows.

## Code Examples

**Complete example combining fakes, mocks, fixtures, and AAA pattern**

```python
import pytest
from unittest.mock import Mock
from dataclasses import dataclass

@dataclass
class Product:
    id: int
    name: str
    price: float

class FakeProductRepo:
    """Fake: a simplified working implementation."""
    def __init__(self):
        self._products: dict[int, Product] = {}

    def save(self, product: Product) -> None:
        self._products[product.id] = product

    def find(self, product_id: int) -> Product | None:
        return self._products.get(product_id)

    def all(self) -> list[Product]:
        return list(self._products.values())

class ProductService:
    def __init__(self, repo, notifier):
        self.repo = repo
        self.notifier = notifier

    def add_product(self, product: Product) -> None:
        self.repo.save(product)
        self.notifier.notify(f'New product: {product.name}')

@pytest.fixture
def repo():
    return FakeProductRepo()

@pytest.fixture
def notifier():
    return Mock()  # Mock: we will verify interactions

@pytest.fixture
def service(repo, notifier):
    return ProductService(repo, notifier)

def test_add_product_stores_in_repo(service, repo):
    # Arrange
    product = Product(id=1, name='Widget', price=9.99)

    # Act
    service.add_product(product)

    # Assert (using fake repo to verify state)
    assert repo.find(1) == product
    assert len(repo.all()) == 1

def test_add_product_sends_notification(service, notifier):
    # Arrange
    product = Product(id=2, name='Gadget', price=19.99)

    # Act
    service.add_product(product)

    # Assert (using mock notifier to verify interaction)
    notifier.notify.assert_called_once_with('New product: Gadget')
```

**Test isolation with monkeypatch and tmp_path fixtures**

```python
import pytest
from unittest.mock import patch

def load_config() -> dict:
    import os
    return {
        'debug': os.environ.get('DEBUG', 'false') == 'true',
        'db_url': os.environ.get('DATABASE_URL', 'sqlite:///:memory:'),
    }

def test_config_debug_mode(monkeypatch):
    """monkeypatch ensures env changes are reverted after the test."""
    monkeypatch.setenv('DEBUG', 'true')
    config = load_config()
    assert config['debug'] is True

def test_config_default_mode():
    """This test is unaffected by the monkeypatch above."""
    config = load_config()
    assert config['debug'] is False

def test_write_report(tmp_path):
    """tmp_path provides an isolated temporary directory per test."""
    report_file = tmp_path / 'report.txt'
    report_file.write_text('Test results: all passed')
    assert report_file.read_text() == 'Test results: all passed'
```


## Resources

- [pytest fixtures documentation](https://docs.pytest.org/en/stable/how-to/fixtures.html) — Official pytest guide to fixtures, scopes, and teardown
- [unittest.mock - Python Docs](https://docs.python.org/3/library/unittest.mock.html) — Standard library mock module reference
- [pytest monkeypatch](https://docs.pytest.org/en/stable/how-to/monkeypatch.html) — Guide to safely modifying environment and objects during tests

---

> 📘 *This lesson is part of the [Python Architecture: Patterns & Type System](https://stanza.dev/courses/python-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*