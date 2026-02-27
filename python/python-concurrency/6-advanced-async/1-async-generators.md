---
source_course: "python-concurrency"
source_lesson: "python-concurrency-async-generators"
---

# Async Generators

## Introduction
Async generators combine `async def` with `yield` to produce values lazily from asynchronous data sources. They are the async equivalent of regular generators: instead of loading all data into memory at once, they yield items one at a time as async operations complete. This lesson covers basic async generators, paginated API streaming, async comprehensions, and cleanup patterns.

## Key Concepts
- **Async generator**: A function defined with `async def` that contains `yield` expressions. It produces values lazily via `async for`.
- **async for**: A loop construct that consumes an async generator or async iterator, awaiting each value.
- **Async comprehension**: List, set, or dict comprehensions using `async for` to consume async iterables.
- **finally in async generators**: Cleanup code in a `try/finally` block runs when the generator is closed or exhausted.

## Real World Context
A log monitoring tool streams events from a remote API that returns paginated results. An async generator fetches each page, yields individual log entries, and handles pagination transparently. The consumer processes entries one at a time with constant memory usage, even when the total dataset is millions of entries. When the consumer stops early, the `finally` block closes the HTTP connection.

## Deep Dive

### Basic Async Generator

```python
async def async_range(n):
    for i in range(n):
        await asyncio.sleep(0.1)  # Async operation
        yield i

async def main():
    async for num in async_range(5):
        print(num)
```

### Paginated API Example

```python
async def fetch_all_pages(base_url):
    page = 1
    while True:
        response = await fetch(f"{base_url}?page={page}")
        data = response.json()
        
        if not data['items']:
            break
        
        for item in data['items']:
            yield item
        
        page += 1

async def main():
    async for user in fetch_all_pages("/api/users"):
        process(user)
```

### Async Comprehensions

```python
# Async list comprehension
results = [item async for item in async_generator()]

# With condition
filtered = [x async for x in source if x > 0]

# Async generator expression
gen = (x * 2 async for x in async_range(10))
```

### Handling Cleanup

```python
async def stream_data():
    connection = await connect()
    try:
        while True:
            data = await connection.read()
            if not data:
                break
            yield data
    finally:
        await connection.close()
```

## Common Pitfalls
1. **Using a list comprehension to collect all results** — Writing `[item async for item in generator()]` loads everything into memory, defeating the purpose of lazy streaming. Only collect all results if you truly need them.
2. **Not using try/finally for cleanup** — If a consumer breaks out of an `async for` loop early, the generator is closed. Without a `finally` block, resources like connections and file handles leak.
3. **Mixing yield and return with values** — An async generator can use `return` to stop iteration, but it cannot return a value. Writing `return data` inside an async generator raises a `SyntaxError`.

## Best Practices
1. **Use async generators for paginated APIs** — They encapsulate pagination logic and present a clean streaming interface to the consumer, keeping concerns separated.
2. **Always wrap resource acquisition in try/finally** — Ensure connections, files, and other resources are cleaned up when the generator exits, whether normally or via early consumer break.

## Summary
- Async generators produce values lazily from async data sources using `async def` with `yield`.
- They are consumed with `async for` loops or async comprehensions.
- Paginated API streaming is a canonical use case: fetch pages internally, yield items externally.
- Always use `try/finally` to clean up resources when an async generator is closed early.
- Async comprehensions provide concise syntax but load all results into memory.

## Code Examples

**Ticker async generator**

```python
import asyncio

async def ticker(interval, count):
    """Yield timestamps at regular intervals."""
    loop = asyncio.get_running_loop()
    for _ in range(count):
        await asyncio.sleep(interval)
        yield loop.time()

async def main():
    async for timestamp in ticker(0.5, 5):
        print(f"Tick at {timestamp:.2f}")

asyncio.run(main())
```


## Resources

- [Async Generators](https://docs.python.org/3.14/reference/expressions.html#asynchronous-generator-functions) — Asynchronous generator functions combining async def with yield

---

> 📘 *This lesson is part of the [Python Concurrency: Asyncio & Free-Threading](https://stanza.dev/courses/python-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*