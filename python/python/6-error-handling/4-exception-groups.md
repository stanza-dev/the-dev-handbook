---
source_course: "python"
source_lesson: "python-exception-groups"
---

# Exception Groups (Python 3.11+)

## Introduction
When multiple concurrent tasks fail at the same time, a single `except` clause is not enough. Exception Groups, introduced in Python 3.11, bundle multiple exceptions into one and provide `except*` to handle each type independently. This is essential for modern async Python.

## Key Concepts
- **`ExceptionGroup`**: A container that holds multiple exceptions, raised when several failures occur simultaneously.
- **`except*`**: A handler that matches exceptions within a group by type, potentially running multiple handlers.
- **Re-raising unmatched exceptions**: Any exceptions in the group not matched by an `except*` clause are automatically re-raised in a new group.

## Real World Context
Async task groups (`asyncio.TaskGroup`) raise `ExceptionGroup` when multiple tasks fail concurrently. Without `except*`, you would need to manually iterate over the group's exceptions. This pattern also applies to parallel HTTP requests, batch database operations, and any scenario where multiple independent operations can each fail.

## Deep Dive

### Why Exception Groups?

Concurrent operations can raise multiple exceptions at once:

```python
async def main():
    async with asyncio.TaskGroup() as tg:
        tg.create_task(may_fail_1())
        tg.create_task(may_fail_2())
        tg.create_task(may_fail_3())
    # If multiple tasks fail, ExceptionGroup is raised
```

### Creating Exception Groups

```python
errors = [
    ValueError("Invalid value"),
    TypeError("Wrong type"),
    KeyError("Missing key")
]
raise ExceptionGroup("Multiple errors occurred", errors)
```

### Catching with except*

```python
try:
    raise ExceptionGroup("errors", [
        ValueError("bad value"),
        TypeError("bad type"),
        KeyError("missing")
    ])
except* ValueError as eg:
    print(f"Value errors: {eg.exceptions}")
except* TypeError as eg:
    print(f"Type errors: {eg.exceptions}")
# KeyError will re-raise as it's not caught
```

### Key Differences from Regular except

- `except*` can match multiple times (one per exception type)
- Unmatched exceptions are re-raised in a new group
- Cannot mix `except` and `except*` in the same `try`

```python
# This is INVALID:
try:
    ...
except ValueError:  # Regular except
    ...
except* TypeError:  # except* -- cannot mix!
    ...
```

## Common Pitfalls
1. **Mixing `except` and `except*` in the same `try` block** -- Python does not allow it. If any handler uses `except*`, all handlers in that block must use `except*`.
2. **Assuming `except*` catches only one exception** -- The variable bound by `except* ValueError as eg` contains an `ExceptionGroup` with potentially multiple `ValueError` instances. Always iterate `eg.exceptions`.
3. **Forgetting that unmatched exceptions re-raise** -- If your `except*` clauses do not cover all types in the group, the remaining exceptions propagate automatically. This is intentional but can surprise you if you expected full coverage.

## Best Practices
1. **Use `asyncio.TaskGroup` instead of `gather(return_exceptions=True)`** -- TaskGroup raises `ExceptionGroup` on failure, making it easy to handle errors per type with `except*`.
2. **Handle each exception type separately** -- Write one `except*` clause per type rather than catching a broad base class, so each error gets appropriate treatment.

## Summary
- `ExceptionGroup` bundles multiple exceptions from concurrent failures into a single raisable object.
- `except*` handlers match exceptions within the group by type; multiple handlers can fire for the same group.
- You cannot mix `except` and `except*` in the same `try` block.
- Unmatched exceptions are automatically re-raised in a new group.
- This pattern is essential for `asyncio.TaskGroup` and any concurrent error-handling scenario.

## Code Examples

**Exception groups with asyncio**

```python
# Practical: handling concurrent task failures
import asyncio

async def risky_task(n):
    if n == 2:
        raise ValueError(f"Task {n} failed")
    if n == 3:
        raise TypeError(f"Task {n} wrong type")
    return n

async def main():
    try:
        async with asyncio.TaskGroup() as tg:
            tasks = [tg.create_task(risky_task(i)) for i in range(5)]
    except* ValueError as eg:
        print(f"Value errors: {[str(e) for e in eg.exceptions]}")
    except* TypeError as eg:
        print(f"Type errors: {[str(e) for e in eg.exceptions]}")
```


## Resources

- [ExceptionGroup](https://docs.python.org/3.14/library/exceptions.html#ExceptionGroup) — Official Python 3.14 reference for ExceptionGroup and the except* syntax

---

> 📘 *This lesson is part of the [Python Fundamentals: Modern 3.14 Edition](https://stanza.dev/courses/python) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*