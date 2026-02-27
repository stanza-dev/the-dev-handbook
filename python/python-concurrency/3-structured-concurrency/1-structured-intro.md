---
source_course: "python-concurrency"
source_lesson: "python-concurrency-structured-intro"
---

# What is Structured Concurrency?

## Introduction
Structured concurrency is a paradigm that ties the lifetime of concurrent tasks to their lexical scope, guaranteeing that no task outlives its creator. Before structured concurrency, fire-and-forget tasks could silently fail, leak resources, or outlive the function that spawned them. This lesson explains the problem, the guarantees structured concurrency provides, and how Python implements it.

## Key Concepts
- **Structured concurrency**: A model where concurrent tasks have clear lifetimes bound to a scope. When the scope exits, all tasks are guaranteed to be complete or cancelled.
- **Orphan task**: A task that outlives the function or scope that created it, making error handling and cleanup unpredictable.
- **TaskGroup**: Python's implementation of structured concurrency (3.11+), an async context manager that owns all tasks created within it.

## Real World Context
A payment processing function spawns three concurrent tasks: charge the card, send a confirmation email, and update inventory. Without structured concurrency, if the charge fails, the email and inventory tasks might still run, leading to inconsistent state. With a TaskGroup, a failure in any task automatically cancels the others and propagates the error, keeping the system consistent.

## Deep Dive

### The Problem with Unstructured Concurrency

```python
# Unstructured: Tasks can outlive their creators
async def risky():
    task = asyncio.create_task(background_work())
    return "done"  # background_work might still be running!

# What if background_work fails after risky() returns?
# What if the program exits before it completes?
```

### Structured Concurrency Guarantees

1. **Task lifetime bound to scope** - All tasks complete before scope exits
2. **Error propagation** - Failures in child tasks propagate to parent
3. **Automatic cancellation** - If parent fails, children are cancelled
4. **No orphan tasks** - Tasks can't outlive their spawning scope

### Before TaskGroup (Python < 3.11)

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

### With TaskGroup (Python 3.11+)

```python
async def main():
    async with asyncio.TaskGroup() as tg:
        tg.create_task(task1())
        tg.create_task(task2())
        tg.create_task(task3())
    # Guaranteed: All tasks complete or are cancelled
    # before we get here
```

## Common Pitfalls
1. **Mixing structured and unstructured concurrency** — Creating tasks with `asyncio.create_task()` inside a TaskGroup bypasses the group's lifecycle management. Always use `tg.create_task()` within a TaskGroup.
2. **Assuming gather provides structured concurrency** — `asyncio.gather()` does not cancel other tasks when one fails (unless you explicitly handle it). Tasks can continue running after an exception.

## Best Practices
1. **Default to TaskGroup for new async code** — It provides automatic cancellation, proper error propagation, and no orphan tasks. Use `gather()` only when you need backwards compatibility with Python < 3.11.
2. **Keep TaskGroup scopes small and focused** — A TaskGroup that spans an entire request handler is harder to reason about than several small, focused groups.

## Summary
- Structured concurrency binds task lifetimes to their lexical scope, preventing orphan tasks.
- It guarantees error propagation, automatic cancellation of sibling tasks, and complete cleanup before the scope exits.
- Python implements structured concurrency via `asyncio.TaskGroup` (3.11+).
- `asyncio.gather()` does not provide these guarantees and should be considered a legacy API for new code.

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


## Resources

- [TaskGroup](https://docs.python.org/3.14/library/asyncio-task.html#asyncio.TaskGroup) — Structured concurrency with asyncio TaskGroup

---

> 📘 *This lesson is part of the [Python Concurrency: Asyncio & Free-Threading](https://stanza.dev/courses/python-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*