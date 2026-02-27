---
source_course: "python-concurrency"
source_lesson: "python-concurrency-async-context-managers"
---

# Async Context Managers

## Introduction
Async context managers bring the safety of `with` statements to asynchronous resource management. They ensure that connections, transactions, file handles, and other async resources are properly opened and closed, even when exceptions occur. This lesson covers the async context manager protocol, the `@asynccontextmanager` decorator, combining multiple managers, and `AsyncExitStack` for dynamic scenarios.

## Key Concepts
- **Async context manager**: An object implementing `__aenter__()` and `__aexit__()` as async methods, used with `async with`.
- **@asynccontextmanager**: A decorator from `contextlib` that creates an async context manager from a generator function with a single `yield`.
- **AsyncExitStack**: A tool for managing a dynamic number of async context managers, useful when the number of resources is not known at compile time.

## Real World Context
A web request handler needs a database connection, a Redis cache connection, and an HTTP session for making external API calls. Each resource requires async setup and teardown. Using `async with` for all three ensures they are properly closed in reverse order, even if the handler raises an exception. For a file processing tool that opens a variable number of files, `AsyncExitStack` manages them all.

## Deep Dive

### The Protocol

```python
class AsyncContextManager:
    async def __aenter__(self):
        # Async setup
        return self
    
    async def __aexit__(self, exc_type, exc_val, exc_tb):
        # Async cleanup
        pass
```

### Database Connection Example

```python
class AsyncDatabase:
    async def __aenter__(self):
        self.connection = await asyncpg.connect(DSN)
        return self.connection
    
    async def __aexit__(self, exc_type, exc_val, exc_tb):
        await self.connection.close()

async def main():
    async with AsyncDatabase() as conn:
        result = await conn.fetch("SELECT * FROM users")
```

### Using @asynccontextmanager

```python
from contextlib import asynccontextmanager

@asynccontextmanager
async def transaction(connection):
    tx = await connection.begin()
    try:
        yield tx
        await tx.commit()
    except Exception:
        await tx.rollback()
        raise

async def main():
    async with AsyncDatabase() as conn:
        async with transaction(conn) as tx:
            await tx.execute("INSERT ...")
```

### Combining Multiple Context Managers

```python
async def main():
    async with (
        AsyncDatabase() as db,
        AsyncCache() as cache,
        AsyncLogger() as logger
    ):
        # All resources available
        pass
```

### ExitStack for Dynamic Context Managers

```python
from contextlib import AsyncExitStack

async def process_files(filenames):
    async with AsyncExitStack() as stack:
        files = [
            await stack.enter_async_context(aiofiles.open(f))
            for f in filenames
        ]
        # All files automatically closed on exit
```

## Common Pitfalls
1. **Raising exceptions in `__aexit__` that mask the original error** — If `__aexit__` raises an exception during cleanup, it replaces the original exception from the `async with` body. Use `try/except` inside `__aexit__` to log cleanup errors without masking the original.
2. **Forgetting to yield in @asynccontextmanager** — The decorated function must contain exactly one `yield`. Omitting it causes a confusing `RuntimeError`.
3. **Not handling exceptions in the @asynccontextmanager** — If you do not wrap `yield` in `try/except`, exceptions from the body will not trigger cleanup code like rollbacks.

## Best Practices
1. **Use @asynccontextmanager for simple cases** — The decorator is more concise than implementing a full class with `__aenter__` and `__aexit__`. Reserve the class approach for complex stateful managers.
2. **Use AsyncExitStack for dynamic resource management** — When the number of resources varies at runtime (e.g., opening N files), `AsyncExitStack` is cleaner than nested `async with` blocks.

## Summary
- Async context managers ensure proper async setup and teardown of resources via `async with`.
- The protocol requires `__aenter__()` and `__aexit__()` async methods.
- `@asynccontextmanager` creates lightweight async context managers from generator functions.
- Multiple async context managers can be combined in a single `async with` statement (Python 3.10+).
- `AsyncExitStack` manages a dynamic number of async resources cleanly.

## Code Examples

**Timeout context manager**

```python
from contextlib import asynccontextmanager
import asyncio

@asynccontextmanager
async def timeout_context(seconds):
    """Timeout context manager."""
    try:
        async with asyncio.timeout(seconds):
            yield
    except asyncio.TimeoutError:
        print(f"Operation timed out after {seconds}s")
        raise

async def main():
    try:
        async with timeout_context(5):
            await slow_operation()
    except asyncio.TimeoutError:
        print("Handling timeout...")
```


## Resources

- [Async Context Managers](https://docs.python.org/3.14/library/contextlib.html#asynccontextmanager) — Creating async context managers with @asynccontextmanager

---

> 📘 *This lesson is part of the [Python Concurrency: Asyncio & Free-Threading](https://stanza.dev/courses/python-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*