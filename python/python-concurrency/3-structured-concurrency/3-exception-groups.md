---
source_course: "python-concurrency"
source_lesson: "python-concurrency-exception-groups"
---

# Exception Groups for Concurrency

## Why Exception Groups?

Concurrent operations can produce multiple errors simultaneously. Traditional exception handling can only handle one at a time.

```python
# Multiple tasks can fail at once
async def main():
    async with asyncio.TaskGroup() as tg:
        tg.create_task(failing_task_1())  # ValueError
        tg.create_task(failing_task_2())  # TypeError
        tg.create_task(failing_task_3())  # KeyError
    # How to handle all three errors?
```

## The except* Syntax

```python
try:
    async with asyncio.TaskGroup() as tg:
        tg.create_task(task1())  # Raises ValueError
        tg.create_task(task2())  # Raises TypeError
except* ValueError as eg:
    # eg is an ExceptionGroup containing ValueErrors
    for exc in eg.exceptions:
        print(f"ValueError: {exc}")
except* TypeError as eg:
    for exc in eg.exceptions:
        print(f"TypeError: {exc}")
```

## Working with ExceptionGroups

```python
from exceptiongroup import ExceptionGroup

# Manual creation
eg = ExceptionGroup("multiple errors", [
    ValueError("bad value"),
    TypeError("wrong type"),
])

# Filtering
value_errors, rest = eg.split(ValueError)
print(value_errors.exceptions)  # [ValueError(...)]
print(rest.exceptions)          # [TypeError(...)]

# Subgroup by predicate
def is_critical(exc):
    return "critical" in str(exc)

critical, other = eg.split(is_critical)
```

## Best Practices

```python
async def robust_operation():
    try:
        async with asyncio.TaskGroup() as tg:
            tg.create_task(operation1())
            tg.create_task(operation2())
    except* RecoverableError:
        # Handle recoverable errors
        return default_value
    except* CriticalError as eg:
        # Log and re-raise critical errors
        for exc in eg.exceptions:
            log.error(f"Critical failure: {exc}")
        raise
```

## Code Examples

**Handling partial failures**

```python
# Practical: Fetch multiple URLs, handle partial failures
async def fetch_all(urls):
    results = {}
    errors = []
    
    try:
        async with asyncio.TaskGroup() as tg:
            tasks = {tg.create_task(fetch(url)): url for url in urls}
    except* Exception as eg:
        # Collect errors but continue
        for exc in eg.exceptions:
            errors.append(exc)
    
    # Get successful results
    for task, url in tasks.items():
        if not task.cancelled() and task.exception() is None:
            results[url] = task.result()
    
    return results, errors
```


---

> 📘 *This lesson is part of the [Python Concurrency: Asyncio & No-GIL](https://stanza.dev/courses/python-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*