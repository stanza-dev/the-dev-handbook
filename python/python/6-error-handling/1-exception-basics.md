---
source_course: "python"
source_lesson: "python-exception-basics"
---

# Exception Fundamentals

## Introduction
Exceptions are Python's mechanism for signaling and handling errors. Understanding the try/except/else/finally block, the exception hierarchy, and Python 3.14's bracketless except syntax gives you precise control over error handling in any application.

## Key Concepts
- **`try` / `except`**: Execute code and catch specific errors.
- **`else`**: Runs only when no exception was raised in the `try` block.
- **`finally`**: Runs unconditionally, for cleanup like closing files or connections.
- **Exception hierarchy**: A tree of built-in exceptions rooted at `BaseException`, with `Exception` as the parent of most catchable errors.
- **Bracketless except (Python 3.14+)**: Catch multiple exception types without parentheses.

## Real World Context
Every production application needs robust error handling. A web server catches `ValueError` on bad input, `ConnectionError` on database failures, and logs unexpected exceptions for debugging. The exception hierarchy matters because catching `Exception` also catches `ValueError`, `KeyError`, and everything under it -- which can mask bugs if you are not careful.

## Deep Dive

### Basic try/except

```python
try:
    result = 10 / 0
except ZeroDivisionError:
    print("Cannot divide by zero")
```

### Multiple Exception Types

```python
try:
    value = int(input())
    result = 10 / value
except ValueError:
    print("Invalid input")
except ZeroDivisionError:
    print("Cannot divide by zero")
except (TypeError, AttributeError) as e:
    print(f"Type error: {e}")
```

### Bracketless except (Python 3.14+)

Python 3.14 allows catching multiple exception types without parentheses:

```python
# Before 3.14 -- parentheses required
try:
    connect()
except (TimeoutError, ConnectionRefusedError):
    print("Network error")

# Python 3.14+ -- no parentheses needed
try:
    connect()
except TimeoutError, ConnectionRefusedError:
    print("Network error")
```

This also works with `except*` for exception groups.

### The Complete try Statement

```python
try:
    # Code that might raise an exception
    result = risky_operation()
except SomeError as e:
    # Handle the exception
    log_error(e)
except Exception as e:
    # Catch-all (use sparingly)
    print(f"Unexpected error: {e}")
else:
    # Runs ONLY if no exception was raised
    process(result)
finally:
    # ALWAYS runs (cleanup)
    cleanup()
```

### Exception Hierarchy

```
BaseException
├── SystemExit
├── KeyboardInterrupt
├── GeneratorExit
└── Exception
    ├── StopIteration
    ├── ArithmeticError
    │   ├── ZeroDivisionError
    │   └── OverflowError
    ├── LookupError
    │   ├── KeyError
    │   └── IndexError
    ├── ValueError
    └── ...
```

## Common Pitfalls
1. **Catching `Exception` too broadly** -- A bare `except Exception:` swallows errors you did not expect, making bugs invisible. Catch the specific type you anticipate.
2. **Using `except:` without a type** -- This catches even `KeyboardInterrupt` and `SystemExit`, making it impossible to stop the program with Ctrl+C. Always specify at least `Exception`.
3. **Putting too much code in `try`** -- Only wrap the specific line(s) that might raise. Extra code in the `try` block risks catching unrelated errors.

## Best Practices
1. **Use `else` for success-path logic** -- Code in `else` only runs when `try` succeeds and will not accidentally catch new exceptions.
2. **Use `finally` for cleanup** -- Closing files, releasing locks, and rolling back transactions belong in `finally` (or better yet, use a context manager).

## Summary
- `try/except` catches exceptions; `else` handles the success path; `finally` always runs for cleanup.
- The exception hierarchy determines which `except` clause catches which errors.
- Python 3.14 introduces bracketless `except` for catching multiple types.
- Always catch specific exception types rather than broad `Exception` or bare `except`.
- Keep `try` blocks minimal to avoid masking unrelated errors.

## Code Examples

**Exception details**

```python
# Catching the exception object
try:
    data = json.loads(invalid_json)
except json.JSONDecodeError as e:
    print(f"Error at line {e.lineno}, column {e.colno}")
    print(f"Message: {e.msg}")

# Bare except (avoid!)
try:
    something()
except:  # Catches EVERYTHING including KeyboardInterrupt
    pass  # Never do this in production
```


## Resources

- [Errors and Exceptions](https://docs.python.org/3.14/tutorial/errors.html) — Official Python 3.14 tutorial on exception handling fundamentals

---

> 📘 *This lesson is part of the [Python Fundamentals: Modern 3.14 Edition](https://stanza.dev/courses/python) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*