---
source_course: "python-concurrency"
source_lesson: "python-concurrency-async-debugging"
---

# Debugging Async Code

## Introduction
Async bugs are notoriously hard to reproduce and diagnose. A forgotten `await` silently discards work, a blocking call freezes everything, and fire-and-forget tasks vanish without a trace. This lesson equips you with asyncio's debug mode, common mistake patterns and their fixes, introspection tools, and the new Python 3.14 CLI introspection commands.

## Key Concepts
- **Debug mode**: A special asyncio mode that detects common mistakes like unawaited coroutines and slow callbacks.
- **Introspection**: The ability to inspect running tasks, their names, and their call stacks at runtime.
- **Call graph**: A representation of which tasks spawned which other tasks, useful for understanding complex async programs.

## Real World Context
Your async web service has an intermittent bug: one in a hundred requests returns stale data. After enabling asyncio debug mode, you discover a coroutine that was never awaited — it was supposed to refresh a cache but silently created a coroutine object that was immediately garbage-collected. Python 3.14's `asyncio ps` command lets you inspect the running tasks of your production process without attaching a debugger.

## Deep Dive

### Debug Mode

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

### Common Mistakes

#### 1. Forgetting to await

```python
# WRONG
async def main():
    fetch_data()  # Coroutine created but never runs!

# RIGHT
async def main():
    await fetch_data()
```

#### 2. Blocking the event loop

```python
# WRONG
async def main():
    time.sleep(5)  # Blocks entire loop

# RIGHT
async def main():
    await asyncio.sleep(5)
```

#### 3. Creating tasks without awaiting

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

### Introspection

```python
# Get all running tasks
for task in asyncio.all_tasks():
    print(task.get_name(), task.get_coro())

# Current running task
current = asyncio.current_task()
```

### Asyncio CLI Introspection (Python 3.14+)

Python 3.14 adds command-line tools to inspect running asyncio programs:

```bash
# Show all running tasks in a process
python -m asyncio ps <PID>

# Show task call tree
python -m asyncio pstree <PID>
```

You can also capture call graphs programmatically:

```python
import asyncio

async def main():
    # Capture the current call graph
    graph = asyncio.capture_call_graph()
    asyncio.print_call_graph(graph)
```

## Common Pitfalls
1. **Forgetting to await a coroutine** — The coroutine object is created but never executed. The function appears to do nothing and Python emits a `RuntimeWarning` that is easy to miss.
2. **Catching Exception too broadly** — Catching `Exception` inside a coroutine will swallow `CancelledError` in Python < 3.11, preventing proper task cancellation. Always re-raise `CancelledError`.
3. **Not using debug mode during development** — Many async bugs are silent. Debug mode catches unawaited coroutines, slow callbacks, and other issues that would otherwise go undetected.

## Best Practices
1. **Enable debug mode in development and CI** — Set `PYTHONASYNCIODEBUG=1` in your development environment to catch issues early.
2. **Name your tasks** — Use `asyncio.create_task(coro(), name="refresh-cache")` so that introspection output is readable and you can identify tasks in logs.
3. **Use Python 3.14 CLI tools for production debugging** — `python -m asyncio ps <PID>` and `pstree` let you inspect running tasks without modifying code or attaching a debugger.

## Summary
- Enable debug mode with `asyncio.run(main(), debug=True)` or `PYTHONASYNCIODEBUG=1` to detect common async mistakes.
- The three most common async bugs are: forgetting to await, blocking the event loop, and fire-and-forget tasks.
- Use `asyncio.all_tasks()` and `asyncio.current_task()` for runtime introspection.
- Python 3.14 adds `asyncio ps`, `pstree`, and `capture_call_graph()` for inspecting async programs.
- Always name your tasks for easier debugging.

## Code Examples

**Task debugging**

```python
import asyncio
import warnings

# Catch 'coroutine was never awaited' warnings
warnings.filterwarnings('error', category=RuntimeWarning)

# Custom task factory for debugging
async def main():
    loop = asyncio.get_running_loop()
    
    def task_factory(loop, coro, **kwargs):
        task = asyncio.Task(coro, **kwargs)
        print(f"Created task: {task.get_name()}")
        return task
    
    loop.set_task_factory(task_factory)
    await some_work()

asyncio.run(main())
```


## Resources

- [Developing with asyncio](https://docs.python.org/3.14/library/asyncio-dev.html) — Debug mode, common pitfalls, and best practices for asyncio development

---

> 📘 *This lesson is part of the [Python Concurrency: Asyncio & Free-Threading](https://stanza.dev/courses/python-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*