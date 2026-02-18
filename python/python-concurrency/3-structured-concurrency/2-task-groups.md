---
source_course: "python-concurrency"
source_lesson: "python-concurrency-task-groups"
---

# asyncio.TaskGroup

TaskGroup provides structured concurrency in Python.

## Basic Usage

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

## Error Handling

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

## Multiple Exception Types

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

## Handling CancelledError

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


---

> 📘 *This lesson is part of the [Python Concurrency: Asyncio & No-GIL](https://stanza.dev/courses/python-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*