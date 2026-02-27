---
source_course: "python-concurrency"
source_lesson: "python-concurrency-task-groups"
---

# TaskGroups (Python 3.11+)

## Introduction
`asyncio.TaskGroup` is Python's built-in implementation of structured concurrency. It provides an async context manager that owns every task created within it and enforces strict lifecycle guarantees: when the scope exits, all tasks are complete; when any task fails, all siblings are cancelled. This lesson covers basic usage, error handling with ExceptionGroups, and graceful cancellation.

## Key Concepts
- **TaskGroup**: An async context manager that manages a set of tasks and enforces structured concurrency.
- **ExceptionGroup**: A container for multiple exceptions, raised by TaskGroup when one or more tasks fail.
- **except* syntax**: A new exception handling syntax (Python 3.11+) for catching specific exception types within an ExceptionGroup.
- **CancelledError**: Raised inside a coroutine when its task is cancelled; should be caught for cleanup and then re-raised.

## Real World Context
An e-commerce checkout flow concurrently validates the shipping address, checks inventory, and verifies the payment method. If inventory check fails, you do not want the payment verification to continue. TaskGroup automatically cancels all remaining tasks when one fails, and the ExceptionGroup lets you handle each failure type separately in the calling code.

## Deep Dive

### Basic Usage

```python
import asyncio

async def fetch(url):
    await asyncio.sleep(1)
    return f"data from {url}"

async def main():
    async with asyncio.TaskGroup() as tg:
        task1 = tg.create_task(fetch("url1"))
        task2 = tg.create_task(fetch("url2"))
        task3 = tg.create_task(fetch("url3"))
    
    # All tasks guaranteed complete here
    print(task1.result(), task2.result(), task3.result())
```

### Error Handling

When any task fails:
1. All other tasks are cancelled
2. TaskGroup waits for cancellations to complete
3. Exceptions are collected into an ExceptionGroup

```python
async def main():
    try:
        async with asyncio.TaskGroup() as tg:
            tg.create_task(succeed())
            tg.create_task(fail())  # Raises ValueError
            tg.create_task(succeed())
    except* ValueError as eg:
        # ExceptionGroup with ValueError inside
        for exc in eg.exceptions:
            print(f"Caught: {exc}")
```

### Multiple Exception Types

```python
try:
    async with asyncio.TaskGroup() as tg:
        tg.create_task(raise_value_error())
        tg.create_task(raise_type_error())
except* ValueError as eg:
    print(f"Value errors: {eg.exceptions}")
except* TypeError as eg:
    print(f"Type errors: {eg.exceptions}")
```

### Handling CancelledError

```python
async def graceful_shutdown():
    try:
        while True:
            await asyncio.sleep(1)
            print("Working...")
    except asyncio.CancelledError:
        print("Cleaning up...")
        await cleanup()
        raise  # Re-raise to signal completion
```

## Common Pitfalls
1. **Swallowing CancelledError** — Catching `CancelledError` without re-raising it prevents the TaskGroup from knowing the task has finished. Always re-raise after cleanup.
2. **Expecting except* to work like except** — `except*` handles a subset of exceptions in the group and can match multiple handlers for the same ExceptionGroup. Treating it like regular `except` leads to missed exceptions.
3. **Trying to add tasks after the group scope has started cancelling** — Once a task in the group fails, the group enters a cancelling state. New tasks added via `tg.create_task()` during cleanup may not behave as expected.

## Best Practices
1. **Use except* to handle different failure types separately** — This lets you recover from expected errors (like network timeouts) while re-raising unexpected ones (like programming errors).
2. **Always re-raise CancelledError after cleanup** — Your coroutine should perform necessary cleanup in a `try/except CancelledError` block, then `raise` to signal it has stopped.

## Summary
- `asyncio.TaskGroup` enforces structured concurrency: all tasks complete or are cancelled before the scope exits.
- When a task fails, all sibling tasks are automatically cancelled and exceptions are collected into an ExceptionGroup.
- Use `except*` to handle different exception types from within an ExceptionGroup.
- Always re-raise `CancelledError` after performing cleanup to cooperate with the cancellation protocol.

## Code Examples

**TaskGroup cancellation behavior**

```python
import asyncio

async def fail_soon():
    await asyncio.sleep(0.1)
    raise ValueError("Task failed!")

async def run_forever():
    try:
        while True:
            await asyncio.sleep(1)
    except asyncio.CancelledError:
        print("I was cancelled!")
        raise

async def main():
    try:
        async with asyncio.TaskGroup() as tg:
            tg.create_task(fail_soon())
            tg.create_task(run_forever())
    except* ValueError:
        print("Handled the failure")
    # Output:
    # I was cancelled!
    # Handled the failure
```


## Resources

- [TaskGroup](https://docs.python.org/3.14/library/asyncio-task.html#asyncio.TaskGroup) — TaskGroup API reference for structured concurrent task management

---

> 📘 *This lesson is part of the [Python Concurrency: Asyncio & Free-Threading](https://stanza.dev/courses/python-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*