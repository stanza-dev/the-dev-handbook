---
source_course: "python"
source_lesson: "python-exception-flow"
---

# Exception-Based Control Flow

## Introduction
In many languages, exceptions are reserved for truly exceptional situations. In Python, they are a first-class control flow mechanism. This lesson explains the EAFP philosophy, how iterators use `StopIteration`, and how `try/except/else/finally` gives you fine-grained control over success and failure paths.

## Key Concepts
- **EAFP (Easier to Ask Forgiveness than Permission)**: Try an operation and handle failure, rather than checking preconditions.
- **LBYL (Look Before You Leap)**: Check conditions before acting -- less Pythonic but sometimes appropriate.
- **`StopIteration`**: The exception that signals the end of an iterator.
- **`try/except/else/finally`**: The full exception handling block with success and cleanup paths.

## Real World Context
The EAFP pattern is everywhere in production Python: accessing dictionary keys, opening files, parsing user input, and connecting to databases. It is also more efficient in the common case because checking for permission (LBYL) can be slower or introduce race conditions -- for example, checking if a file exists and then opening it leaves a window where the file can be deleted between the check and the open.

## Deep Dive

### EAFP vs LBYL

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

### StopIteration for Iterators

```python
it = iter([1, 2, 3])

while True:
    try:
        value = next(it)
        print(value)
    except StopIteration:
        break
```

### Using `else` with `try`

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

### Raising Exceptions

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

## Common Pitfalls
1. **Catching too broad an exception** -- Using bare `except:` or `except Exception:` swallows errors you did not anticipate, making debugging much harder. Catch the specific exception type you expect.
2. **Putting too much code in the `try` block** -- Only wrap the line that might raise the exception. Extra code inside `try` can catch unrelated errors and mask bugs.
3. **Forgetting to re-raise after logging** -- If you catch an exception just to log it, always `raise` to propagate it. Silencing exceptions leads to silent failures in production.

## Best Practices
1. **Use EAFP when the exceptional case is rare** -- If the key is usually present, `try/except KeyError` is faster than checking `if key in dict` on every call.
2. **Put success-path code in the `else` block** -- This makes it clear what runs only when the `try` block succeeds and avoids accidentally catching new exceptions.

## Summary
- Python's EAFP philosophy favors trying an operation and catching failures over checking preconditions.
- `StopIteration` is used internally by iterators and `for` loops to signal completion.
- The `else` clause on `try` runs only when no exception was raised; `finally` always runs.
- Catch specific exception types, not broad `Exception`, to avoid masking bugs.
- Re-raise exceptions after logging to avoid silent failures.

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


## Resources

- [Errors and Exceptions](https://docs.python.org/3.14/tutorial/errors.html) — Official Python 3.14 tutorial on exception handling and EAFP patterns

---

> 📘 *This lesson is part of the [Python Fundamentals: Modern 3.14 Edition](https://stanza.dev/courses/python) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*