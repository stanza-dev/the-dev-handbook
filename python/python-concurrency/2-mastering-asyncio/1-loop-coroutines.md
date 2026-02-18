---
source_course: "python-concurrency"
source_lesson: "python-concurrency-event-loop-coroutines"
---

# Asyncio Core Concepts

## The Event Loop

The event loop is the core of asyncio. It:
- Runs async tasks
- Handles I/O events
- Schedules callbacks
- Manages subprocesses

```python
import asyncio

# The standard way to run async code
asyncio.run(main())  # Creates loop, runs, cleans up

# Manual loop control (advanced)
loop = asyncio.get_event_loop()
loop.run_until_complete(main())
loop.close()
```

## Coroutines

Coroutines are functions defined with `async def`. They don't execute when called—they return a coroutine object.

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

## Cooperative Multitasking

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


---

> 📘 *This lesson is part of the [Python Concurrency: Asyncio & No-GIL](https://stanza.dev/courses/python-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*