---
source_course: "python-architecture"
source_lesson: "python-architecture-mocking-patching"
---

# Mocking & Patching

## Introduction

When testing code that depends on external systems — databases, HTTP APIs, file systems, clocks — you need a way to replace those dependencies with controlled substitutes. Python's `unittest.mock` module provides `Mock`, `MagicMock`, and `patch` to do exactly this, letting you isolate the code under test and verify how it interacts with its collaborators.

## Key Concepts

- **Mock**: A flexible object that records how it is called and lets you control return values. Any attribute access or method call creates new Mock objects automatically.
- **MagicMock**: A subclass of Mock that pre-implements Python magic methods (`__len__`, `__iter__`, `__getitem__`, etc.), making it compatible with code that uses dunder protocols.
- **patch**: A context manager / decorator that temporarily replaces a target (module attribute, class, function) with a Mock for the duration of a test.
- **spec**: Constrains a Mock to only allow attributes and methods that exist on the real object, catching typos and API drift.
- **side_effect**: Lets a Mock raise exceptions, return different values on successive calls, or delegate to a custom function.

## Real World Context

Every professional Python test suite uses mocking. When testing a payment service, you mock the Stripe API so tests run without making real charges. When testing a notification system, you mock the email sender so tests complete instantly without sending real emails. Django, Flask, and FastAPI projects all rely heavily on `unittest.mock` for test isolation.

## Deep Dive

### Mock Basics

```python
from unittest.mock import Mock

# Create a mock and configure it
db = Mock()
db.get_user.return_value = {'name': 'Alice', 'age': 30}

# Use it as if it were real
user = db.get_user(42)
assert user['name'] == 'Alice'

# Verify interactions
db.get_user.assert_called_once_with(42)
assert db.get_user.call_count == 1
```

### MagicMock for Dunder Methods

```python
from unittest.mock import MagicMock

# MagicMock supports magic methods out of the box
collection = MagicMock()
collection.__len__.return_value = 5
collection.__getitem__.return_value = 'item'

assert len(collection) == 5
assert collection[0] == 'item'

# Iteration
collection.__iter__.return_value = iter(['a', 'b', 'c'])
for item in collection:
    print(item)  # a, b, c
```

### patch: Replacing Dependencies

```python
from unittest.mock import patch
import myapp.services

# As a decorator
@patch('myapp.services.send_email')
def test_signup_sends_welcome_email(mock_send):
    mock_send.return_value = True
    myapp.services.signup('alice@example.com')
    mock_send.assert_called_once_with(
        to='alice@example.com',
        subject='Welcome!'
    )

# As a context manager
def test_signup_handles_email_failure():
    with patch('myapp.services.send_email') as mock_send:
        mock_send.side_effect = ConnectionError('SMTP down')
        result = myapp.services.signup('bob@example.com')
        assert result.email_sent is False
```

### spec: Keeping Mocks Honest

```python
from unittest.mock import Mock

class UserRepository:
    def find_by_id(self, user_id: int) -> dict:
        ...
    def save(self, user: dict) -> None:
        ...

# Without spec: typos go unnoticed
bad_mock = Mock()
bad_mock.find_by_idd(1)  # No error! Silent bug.

# With spec: typos raise AttributeError
good_mock = Mock(spec=UserRepository)
good_mock.find_by_id(1)  # OK
good_mock.find_by_idd(1)  # AttributeError!
```

### side_effect: Dynamic Behavior

```python
from unittest.mock import Mock

api = Mock()

# Raise an exception
api.connect.side_effect = TimeoutError('Connection timed out')

# Return different values on successive calls
api.fetch.side_effect = [{'data': 'first'}, {'data': 'second'}, StopIteration]

# Delegate to a function
def fake_fetch(url):
    if 'users' in url:
        return {'users': []}
    raise ValueError(f'Unknown endpoint: {url}')

api.fetch.side_effect = fake_fetch
```

### Patch Where It Is Used, Not Where It Is Defined

```python
# myapp/services.py
from myapp.email import send_email  # imported here

def signup(email):
    send_email(to=email, subject='Welcome!')

# test_services.py
# WRONG: patching where it is defined
@patch('myapp.email.send_email')  # Does not affect services.py!
def test_wrong(mock): ...

# RIGHT: patching where it is used
@patch('myapp.services.send_email')  # This is what services.py sees
def test_right(mock): ...
```

## Common Pitfalls

1. **Patching the wrong target**: Always patch where the name is *looked up*, not where it is *defined*. If `services.py` does `from email import send`, patch `services.send`, not `email.send`.
2. **Over-mocking**: If you mock everything, your tests prove nothing. Only mock external I/O boundaries.
3. **Forgetting spec**: Without `spec`, mocks accept any attribute access silently, hiding bugs when the real API changes.
4. **Not resetting mocks**: In parameterized tests, call `mock.reset_mock()` between iterations or mocks accumulate call history from previous runs.

## Best Practices

- Always use `spec=RealClass` or `autospec=True` to keep mocks in sync with real interfaces.
- Prefer `patch` as a context manager over decorator form when you need fine-grained control.
- Use `assert_called_once_with()` over `assert_called_with()` to verify call count too.
- Combine with dependency injection: if a class accepts its dependencies in `__init__`, you can pass mocks directly without `patch`.
- Keep the mocking surface small: mock at the boundary (HTTP client, DB driver), not deep inside your code.

## Summary

`unittest.mock` provides Mock for recording interactions, MagicMock for dunder protocol support, and patch for temporarily swapping real objects with test doubles. Use `spec` to keep mocks honest and `side_effect` for dynamic behavior. Always patch where the name is looked up, and prefer mocking at I/O boundaries to keep tests meaningful.

## Code Examples

**Mocking a payment gateway with spec and assertion checks**

```python
from unittest.mock import Mock, patch, MagicMock

# --- Real code ---
class PaymentGateway:
    def charge(self, amount: float, token: str) -> dict:
        """Calls external payment API."""
        ...

class OrderService:
    def __init__(self, gateway: PaymentGateway):
        self.gateway = gateway

    def place_order(self, amount: float, token: str) -> str:
        result = self.gateway.charge(amount, token)
        if result['status'] == 'success':
            return f"Order confirmed: {result['id']}"
        raise ValueError(f"Payment failed: {result['error']}")

# --- Test code ---
def test_place_order_success():
    mock_gateway = Mock(spec=PaymentGateway)
    mock_gateway.charge.return_value = {
        'status': 'success',
        'id': 'txn_abc123'
    }

    service = OrderService(mock_gateway)
    result = service.place_order(29.99, 'tok_visa')

    assert result == 'Order confirmed: txn_abc123'
    mock_gateway.charge.assert_called_once_with(29.99, 'tok_visa')

def test_place_order_failure():
    mock_gateway = Mock(spec=PaymentGateway)
    mock_gateway.charge.return_value = {
        'status': 'failed',
        'error': 'Insufficient funds'
    }

    service = OrderService(mock_gateway)
    try:
        service.place_order(29.99, 'tok_visa')
        assert False, 'Should have raised ValueError'
    except ValueError as e:
        assert 'Insufficient funds' in str(e)
```

**Patching HTTP calls and internal functions**

```python
from unittest.mock import patch, Mock

# --- Module: weather_service.py ---
import requests

def get_temperature(city: str) -> float:
    response = requests.get(f'https://api.weather.com/{city}')
    response.raise_for_status()
    return response.json()['temp']

def get_forecast(city: str) -> str:
    temp = get_temperature(city)
    if temp > 30:
        return 'Hot'
    elif temp > 15:
        return 'Mild'
    return 'Cold'

# --- Tests ---
@patch('weather_service.requests.get')
def test_get_temperature(mock_get):
    mock_response = Mock()
    mock_response.json.return_value = {'temp': 22.5}
    mock_response.raise_for_status.return_value = None
    mock_get.return_value = mock_response

    temp = get_temperature('Paris')
    assert temp == 22.5
    mock_get.assert_called_once_with('https://api.weather.com/Paris')

@patch('weather_service.get_temperature')
def test_get_forecast_hot(mock_temp):
    mock_temp.return_value = 35.0
    assert get_forecast('Dubai') == 'Hot'

@patch('weather_service.get_temperature')
def test_get_forecast_cold(mock_temp):
    mock_temp.return_value = 5.0
    assert get_forecast('Oslo') == 'Cold'
```


## Resources

- [unittest.mock - Python Docs](https://docs.python.org/3/library/unittest.mock.html) — Official reference for Mock, MagicMock, and patch
- [unittest.mock getting started](https://docs.python.org/3/library/unittest.mock-examples.html) — Practical examples and recipes for unittest.mock

---

> 📘 *This lesson is part of the [Python Architecture: Patterns & Type System](https://stanza.dev/courses/python-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*