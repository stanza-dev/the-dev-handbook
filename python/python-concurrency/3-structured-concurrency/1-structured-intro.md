---
source_course: "python-concurrency"
source_lesson: "python-concurrency-structured-intro"
---

# Structured Concurrency

Structured concurrency is a programming paradigm that ensures concurrent operations have clear lifetimes tied to their scopes.

## The Problem with Unstructured Concurrency

```python
# Unstructured: Tasks can outlive their creators
async def risky():
    task = asyncio.create_task(background_work())
    return "done"  # background_work might still be running!

# What if background_work fails after risky() returns?
# What if the program exits before it completes?
```

## Structured Concurrency Guarantees

1. **Task lifetime bound to scope** - All tasks complete before scope exits
2. **Error propagation** - Failures in child tasks propagate to parent
3. **Automatic cancellation** - If parent fails, children are cancelled
4. **No orphan tasks** - Tasks can't outlive their spawning scope

## Before TaskGroup (Python < 3.11)

```python
# gather: Runs tasks but has edge cases
async def main():
    try:
        results = await asyncio.gather(
            task1(),
            task2(),
            task3(),
        )
    except Exception:
        # One failed, but others might still be running!
        pass
```

## With TaskGroup (Python 3.11+)

```python
async def main():
    async with asyncio.TaskGroup() as tg:
        tg.create_task(task1())
        tg.create_task(task2())
        tg.create_task(task3())
    # Guaranteed: All tasks complete or are cancelled
    # before we get here
```

## Code Examples

**Structured vs unstructured**

```python
# Visual comparison
# Unstructured (dangerous)
async def spawn_and_forget():
    asyncio.create_task(work())  # Fire and forget
    return  # Task orphaned!

# Structured (safe)
async def spawn_and_wait():
    async with asyncio.TaskGroup() as tg:
        tg.create_task(work())
    # Task guaranteed complete here
```


---

> 📘 *This lesson is part of the [Python Concurrency: Asyncio & No-GIL](https://stanza.dev/courses/python-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*