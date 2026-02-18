---
source_course: "python-concurrency"
source_lesson: "python-concurrency-async-context-managers"
---

# Async Context Managers

Manage async resources with `async with`.

## The Protocol

```python
class AsyncContextManager:
    async def __aenter__(self):
        # Async setup
        return self
    
    async def __aexit__(self, exc_type, exc_val, exc_tb):
        # Async cleanup
        pass
```

## Database Connection Example

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

## Using @asynccontextmanager

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

## Combining Multiple Context Managers

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

## ExitStack for Dynamic Context Managers

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


---

> 📘 *This lesson is part of the [Python Concurrency: Asyncio & No-GIL](https://stanza.dev/courses/python-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*