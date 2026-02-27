---
source_course: "python-concurrency"
source_lesson: "python-concurrency-event-loop-coroutines"
---

# The Event Loop & Coroutines

## Introduction
At the heart of Python's asyncio library is the event loop, a scheduler that juggles thousands of tasks on a single thread without blocking. Combined with coroutines, it enables you to write code that reads sequentially but executes concurrently. This lesson covers how the event loop works, what coroutines are, and how cooperative multitasking lets multiple tasks make progress.

## Key Concepts
- **Event loop**: The central scheduler that runs async tasks, handles I/O events, schedules callbacks, and manages subprocesses.
- **Coroutine**: A function defined with `async def` that returns a coroutine object when called. It does not execute until awaited or scheduled.
- **await**: The keyword that suspends a coroutine, yielding control back to the event loop so other tasks can run.
- **Cooperative multitasking**: A model where tasks voluntarily yield control at `await` points, rather than being preempted by the OS.

## Real World Context
A web framework like FastAPI or Starlette handles thousands of HTTP requests concurrently using asyncio. Each request is a coroutine that `await`s database queries, file reads, and external API calls. While one request waits for a database response, the event loop serves other requests. This is how a single-threaded Python server can match the throughput of multi-threaded servers in other languages.

## Deep Dive

### The Event Loop

The event loop is the core of asyncio. It runs async tasks, handles I/O events, schedules callbacks, and manages subprocesses.

```python
import asyncio

# The standard way to run async code (recommended)
asyncio.run(main())  # Creates loop, runs, cleans up

# Manual loop control (legacy — avoid in new code)
# Note: asyncio.get_event_loop() raises RuntimeError in Python 3.14
# if no current event loop exists. Use asyncio.run() instead.
```

### Coroutines

Coroutines are functions defined with `async def`. They don't execute when called — they return a coroutine object.

```python
async def fetch_data():
    print("Fetching...")
    await asyncio.sleep(1)  # Suspend here
    return "Data"

# This does NOT run the coroutine!
coro = fetch_data()  # Returns coroutine object

# You must await it or schedule it
result = await coro  # Now it runs
```

### Cooperative Multitasking

Tasks voluntarily yield control at `await` points:

```python
async def task_a():
    print("A: start")
    await asyncio.sleep(1)  # Yields to event loop
    print("A: end")

async def task_b():
    print("B: start")
    await asyncio.sleep(0.5)
    print("B: end")

async def main():
    await asyncio.gather(task_a(), task_b())

# Output:
# A: start
# B: start
# B: end (after 0.5s)
# A: end (after 1s total)
```

## Common Pitfalls
1. **Calling a coroutine without awaiting it** — Writing `fetch_data()` without `await` returns a coroutine object that never executes. Python will emit a `RuntimeWarning: coroutine was never awaited`.
2. **Using `asyncio.get_event_loop()` in modern code** — In Python 3.10+, prefer `asyncio.run()` for the top-level entry point. Manual loop management is error-prone and rarely needed.

## Best Practices
1. **Use `asyncio.run()` as your single entry point** — It creates the event loop, runs your main coroutine, and cleans everything up. Avoid creating or managing loops manually.
2. **Think of `await` as a yield point** — Every `await` is an opportunity for other tasks to run. Design your code so long-running sections have regular `await` points.

## Summary
- The event loop is asyncio's scheduler, running tasks, handling I/O, and managing callbacks on a single thread.
- Coroutines are defined with `async def` and must be awaited or scheduled to execute.
- Cooperative multitasking works because tasks voluntarily yield control at every `await` point.
- Use `asyncio.run()` as the standard entry point for async programs.

## Code Examples

**Basic asyncio program**

```python
import asyncio

async def say_hello():
    print("Hello")
    await asyncio.sleep(1)  # Non-blocking wait
    print("World")

# Entry point for asyncio programs
if __name__ == "__main__":
    asyncio.run(say_hello())
```


## Resources

- [asyncio Module](https://docs.python.org/3.14/library/asyncio.html) — Official documentation for Python's asyncio library

---

> 📘 *This lesson is part of the [Python Concurrency: Asyncio & Free-Threading](https://stanza.dev/courses/python-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*