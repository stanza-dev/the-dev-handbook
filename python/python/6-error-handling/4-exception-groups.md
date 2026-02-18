---
source_course: "python"
source_lesson: "python-exception-groups"
---

# Exception Groups

Python 3.11 introduced `ExceptionGroup` for handling multiple exceptions simultaneously.

## Why Exception Groups?

Concurrent operations can raise multiple exceptions at once:

```python
async def main():
    async with asyncio.TaskGroup() as tg:
        tg.create_task(may_fail_1())
        tg.create_task(may_fail_2())
        tg.create_task(may_fail_3())
    # If multiple tasks fail, ExceptionGroup is raised
```

## Creating Exception Groups

```python
errors = [
    ValueError("Invalid value"),
    TypeError("Wrong type"),
    KeyError("Missing key")
]
raise ExceptionGroup("Multiple errors occurred", errors)
```

## Catching with except*

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

## Key Differences from Regular except

- `except*` can match multiple times (one per exception type)
- Unmatched exceptions are re-raised in a new group
- Cannot mix `except` and `except*` in the same `try`

```python
# This is INVALID:
try:
    ...
except ValueError:  # Regular except
    ...
except* TypeError:  # except* - cannot mix!
    ...
```

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


---

> 📘 *This lesson is part of the [Python Fundamentals: Modern 3.15 Edition](https://stanza.dev/courses/python) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*