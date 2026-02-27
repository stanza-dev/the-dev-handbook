---
source_course: "python"
source_lesson: "python-raising-exceptions"
---

# Raising & Creating Exceptions

## Introduction
Knowing how to raise, re-raise, chain, and define custom exceptions is essential for writing libraries and applications that communicate errors clearly. This lesson covers the `raise` statement, exception chaining, custom exception hierarchies, and the role of `assert`.

## Key Concepts
- **`raise`**: Throws an exception, stopping normal execution.
- **Re-raising (`raise` with no argument)**: Propagates the currently handled exception after logging or partial handling.
- **Exception chaining (`from`)**: Links a new exception to its root cause via `__cause__`.
- **Custom exceptions**: Application-specific exception classes that inherit from `Exception`.
- **`assert`**: A debugging aid that raises `AssertionError` when a condition is false (can be disabled with `-O`).

## Real World Context
Well-designed exception hierarchies let callers decide how granularly to handle errors. A library might raise `DatabaseError` as a base, with `ConnectionError` and `QueryError` as specifics. Callers who want a blanket handler catch `DatabaseError`; those who need precision catch the subclass. Exception chaining preserves the full error trail, which is invaluable for debugging issues in production.

## Deep Dive

### The raise Statement

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

### Re-raising Exceptions

```python
try:
    process()
except ValueError:
    log("Processing failed")
    raise  # Re-raise the same exception
```

### Exception Chaining

```python
try:
    config = load_config()
except FileNotFoundError as e:
    raise ConfigError("Config file missing") from e

# The original exception is preserved in __cause__
```

### Custom Exceptions

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

### assert Statements

```python
def calculate_discount(price, percent):
    assert 0 <= percent <= 100, "Percent must be 0-100"
    assert price > 0, "Price must be positive"
    return price * (1 - percent / 100)

# Note: asserts can be disabled with -O flag
```

## Common Pitfalls
1. **Using `assert` for input validation** -- Assertions are disabled when Python runs with `-O` (optimize). Never use `assert` to validate user input or function arguments in production; use `raise ValueError` instead.
2. **Forgetting `from e` when chaining exceptions** -- Without `from e`, the original traceback is lost, making it harder to find the root cause. Always chain with `from` when wrapping exceptions.
3. **Creating custom exceptions that do not call `super().__init__()`** -- Skipping the parent initializer can break pickling and string representation. Always call `super().__init__(message)`.

## Best Practices
1. **Build a custom exception hierarchy for your library** -- A single base exception (e.g., `AppError`) lets callers catch everything from your library with one clause while still being able to handle specifics.
2. **Always use `raise` without arguments to re-raise** -- `raise` alone preserves the original traceback. `raise e` resets it, losing context.

## Summary
- Use `raise` to signal errors explicitly with clear messages.
- Re-raise with bare `raise` to preserve the full traceback.
- Chain exceptions with `from` to link new errors to their root cause.
- Build custom exception hierarchies for libraries and applications.
- Never use `assert` for production input validation -- it can be disabled.

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


## Resources

- [Raising Exceptions](https://docs.python.org/3.14/tutorial/errors.html#raising-exceptions) — Official Python 3.14 tutorial on raising, re-raising, and chaining exceptions

---

> 📘 *This lesson is part of the [Python Fundamentals: Modern 3.14 Edition](https://stanza.dev/courses/python) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*