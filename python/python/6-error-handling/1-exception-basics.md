---
source_course: "python"
source_lesson: "python-exception-basics"
---

# Exception Handling

Exceptions are Python's way of handling errors and exceptional conditions.

## Basic try/except

```python
try:
    result = 10 / 0
except ZeroDivisionError:
    print("Cannot divide by zero")
```

## Multiple Exception Types

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

## The Complete try Statement

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

## Exception Hierarchy

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


---

> 📘 *This lesson is part of the [Python Fundamentals: Modern 3.15 Edition](https://stanza.dev/courses/python) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*