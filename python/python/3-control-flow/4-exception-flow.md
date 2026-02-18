---
source_course: "python"
source_lesson: "python-exception-flow"
---

# Exceptions in Control Flow

Python uses exceptions not just for errors, but as a control flow mechanism.

## EAFP vs LBYL

Python favors **EAFP** (Easier to Ask Forgiveness than Permission) over **LBYL** (Look Before You Leap).

```python
# LBYL (non-Pythonic)
if key in dictionary:
    value = dictionary[key]
else:
    value = default

# EAFP (Pythonic)
try:
    value = dictionary[key]
except KeyError:
    value = default

# Even better
value = dictionary.get(key, default)
```

## StopIteration for Iterators

```python
it = iter([1, 2, 3])

while True:
    try:
        value = next(it)
        print(value)
    except StopIteration:
        break
```

## Using `else` with `try`

```python
try:
    result = risky_operation()
except SomeError:
    handle_error()
else:
    # Only runs if no exception
    process(result)
finally:
    # Always runs
    cleanup()
```

## Raising Exceptions

```python
def divide(a, b):
    if b == 0:
        raise ValueError("Cannot divide by zero")
    return a / b

# Re-raising
try:
    something()
except Exception:
    log_error()
    raise  # Re-raises the caught exception

# Exception chaining
try:
    something()
except OriginalError as e:
    raise NewError("Context") from e
```

## Code Examples

**Custom exception classes**

```python
# Custom exceptions
class ValidationError(Exception):
    def __init__(self, field, message):
        self.field = field
        self.message = message
        super().__init__(f"{field}: {message}")

# Using it
try:
    if not email:
        raise ValidationError("email", "Required field")
except ValidationError as e:
    print(f"Validation failed: {e.field} - {e.message}")
```


---

> 📘 *This lesson is part of the [Python Fundamentals: Modern 3.15 Edition](https://stanza.dev/courses/python) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*