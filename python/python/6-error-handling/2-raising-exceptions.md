---
source_course: "python"
source_lesson: "python-raising-exceptions"
---

# Raising Exceptions

## The raise Statement

```python
def divide(a, b):
    if b == 0:
        raise ValueError("Division by zero is not allowed")
    return a / b

def validate_age(age):
    if age < 0:
        raise ValueError("Age cannot be negative")
    if age > 150:
        raise ValueError("Age seems unrealistic")
```

## Re-raising Exceptions

```python
try:
    process()
except ValueError:
    log("Processing failed")
    raise  # Re-raise the same exception
```

## Exception Chaining

```python
try:
    config = load_config()
except FileNotFoundError as e:
    raise ConfigError("Config file missing") from e

# The original exception is preserved in __cause__
```

## Custom Exceptions

```python
class ValidationError(Exception):
    """Raised when validation fails."""
    
    def __init__(self, field, message):
        self.field = field
        self.message = message
        super().__init__(f"{field}: {message}")

class DatabaseError(Exception):
    """Base class for database errors."""
    pass

class ConnectionError(DatabaseError):
    pass

class QueryError(DatabaseError):
    pass
```

## assert Statements

```python
def calculate_discount(price, percent):
    assert 0 <= percent <= 100, "Percent must be 0-100"
    assert price > 0, "Price must be positive"
    return price * (1 - percent / 100)

# Note: asserts can be disabled with -O flag
```

## Code Examples

**Custom exception hierarchy**

```python
# Best practice: custom exception hierarchy
class AppError(Exception):
    """Base exception for our application."""
    pass

class UserNotFoundError(AppError):
    def __init__(self, user_id):
        self.user_id = user_id
        super().__init__(f"User {user_id} not found")

class PermissionDeniedError(AppError):
    def __init__(self, action, resource):
        super().__init__(f"Cannot {action} on {resource}")
```


---

> 📘 *This lesson is part of the [Python Fundamentals: Modern 3.15 Edition](https://stanza.dev/courses/python) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*