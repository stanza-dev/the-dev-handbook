---
source_course: "python-concurrency"
source_lesson: "python-concurrency-exception-groups"
---

# Exception Groups in Concurrent Code

## Introduction
When multiple tasks run concurrently, multiple failures can happen at the same time. Traditional `try/except` can only handle one exception per block. ExceptionGroups (Python 3.11+) solve this by collecting multiple exceptions into a single object, and the `except*` syntax lets you handle each type selectively. This lesson covers why ExceptionGroups exist, how to use them, and how to split and filter them.

## Key Concepts
- **ExceptionGroup**: A built-in exception type that wraps a list of exceptions, allowing multiple concurrent errors to be represented as one.
- **except* syntax**: A handler that matches and extracts specific exception types from an ExceptionGroup without consuming the rest.
- **split()**: A method on ExceptionGroup that separates matching exceptions from non-matching ones into two subgroups.

## Real World Context
A batch data importer runs 100 concurrent database inserts. Some fail with `IntegrityError` (duplicate keys), others with `ConnectionError` (network blip). With ExceptionGroups, you can catch all `IntegrityError` instances for logging, retry all `ConnectionError` instances, and let unexpected errors propagate, all from a single TaskGroup failure.

## Deep Dive

### Why Exception Groups?

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

### The except* Syntax

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

### Working with ExceptionGroups

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

### Robust Error Handling Pattern

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

## Common Pitfalls
1. **Using except instead of except* for ExceptionGroups** — A regular `except ValueError` will not match a `ValueError` nested inside an ExceptionGroup. You must use `except* ValueError` to unwrap and match.
2. **Forgetting that except* can fire multiple handlers** — Unlike regular except chains where only one handler runs, multiple `except*` blocks can each match different exceptions within the same ExceptionGroup.
3. **Not checking the `rest` value from split()** — When splitting an ExceptionGroup, the non-matching remainder may be `None` if all exceptions matched. Always check before accessing `.exceptions`.

## Best Practices
1. **Separate recoverable from critical errors** — Use `except*` to catch and handle recoverable errors (retries, defaults) while letting critical errors propagate up the call stack.
2. **Use split() for programmatic filtering** — When you need to process exceptions based on custom criteria (not just type), `eg.split(predicate)` gives you fine-grained control.

## Summary
- ExceptionGroups wrap multiple concurrent exceptions into a single object.
- The `except*` syntax selectively matches and handles specific exception types within an ExceptionGroup.
- The `split()` method separates an ExceptionGroup into matching and non-matching subgroups.
- Multiple `except*` handlers can fire for the same ExceptionGroup, unlike regular `except` chains.
- Use ExceptionGroups to distinguish recoverable errors from critical failures in concurrent code.

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


## Resources

- [ExceptionGroup](https://docs.python.org/3.14/library/exceptions.html#ExceptionGroup) — ExceptionGroup and BaseExceptionGroup for handling multiple concurrent errors

---

> 📘 *This lesson is part of the [Python Concurrency: Asyncio & Free-Threading](https://stanza.dev/courses/python-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*