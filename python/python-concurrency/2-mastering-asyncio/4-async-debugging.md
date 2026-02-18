---
source_course: "python-concurrency"
source_lesson: "python-concurrency-async-debugging"
---

# Debugging Asyncio

## Debug Mode

```python
import asyncio

# Enable debug mode
asyncio.run(main(), debug=True)

# Or via environment variable
# PYTHONASYNCIODEBUG=1 python script.py
```

Debug mode provides:
- Warnings for coroutines that were never awaited
- Slow callback detection
- Better error messages

## Common Mistakes

### 1. Forgetting to await

```python
# WRONG
async def main():
    fetch_data()  # Coroutine created but never runs!

# RIGHT
async def main():
    await fetch_data()
```

### 2. Blocking the event loop

```python
# WRONG
async def main():
    time.sleep(5)  # Blocks entire loop

# RIGHT
async def main():
    await asyncio.sleep(5)
```

### 3. Creating tasks without awaiting

```python
# Tasks may not complete!
async def main():
    asyncio.create_task(background_work())
    # main() ends, task may be cancelled

# Better: track and await tasks
async def main():
    task = asyncio.create_task(background_work())
    # ... do other work ...
    await task  # Ensure completion
```

## Introspection

```python
# Get all running tasks
for task in asyncio.all_tasks():
    print(task.get_name(), task.get_coro())

# Current running task
current = asyncio.current_task()
```

## Code Examples

**Task debugging**

```python
import asyncio
import warnings

# Catch 'coroutine was never awaited' warnings
warnings.filterwarnings('error', category=RuntimeWarning)

# Custom task factory for debugging
def task_factory(loop, coro):
    task = asyncio.Task(coro, loop=loop)
    print(f"Created task: {task.get_name()}")
    return task

loop = asyncio.get_event_loop()
loop.set_task_factory(task_factory)
```


---

> 📘 *This lesson is part of the [Python Concurrency: Asyncio & No-GIL](https://stanza.dev/courses/python-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*