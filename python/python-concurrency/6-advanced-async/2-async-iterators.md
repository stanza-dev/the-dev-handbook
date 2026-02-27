---
source_course: "python-concurrency"
source_lesson: "python-concurrency-async-iterators"
---

# Async Iterators

## Introduction
While async generators are the quick way to produce async sequences, async iterators give you full control over the iteration protocol. By implementing `__aiter__` and `__anext__`, you can build custom async data sources like database cursors, WebSocket message streams, and event listeners. This lesson covers the async iterator protocol, practical implementations, the `aiter()`/`anext()` built-ins, and how to merge multiple async streams.

## Key Concepts
- **Async iterator protocol**: An object that implements `__aiter__()` (returns the iterator) and `__anext__()` (an async method that returns the next value or raises `StopAsyncIteration`).
- **StopAsyncIteration**: The exception that signals the end of an async iterator, analogous to `StopIteration` for sync iterators.
- **aiter() / anext()**: Built-in functions (Python 3.10+) for working with async iterators, analogous to `iter()` and `next()`.

## Real World Context
A real-time stock ticker application receives price updates from multiple exchanges via WebSocket. Each exchange is wrapped in an async iterator. A `merge_streams()` function combines them into a single async stream, yielding the latest price from whichever exchange responds first. The consumer processes prices with `async for`, unaware of how many exchanges are feeding the stream.

## Deep Dive

### The Protocol

```python
class AsyncIterator:
    def __aiter__(self):
        return self
    
    async def __anext__(self):
        # Return next value or raise StopAsyncIteration
        pass
```

### Example: Database Cursor

```python
class AsyncCursor:
    def __init__(self, query):
        self.query = query
        self.results = None
        self.index = 0
    
    def __aiter__(self):
        return self
    
    async def __anext__(self):
        if self.results is None:
            self.results = await execute_query(self.query)
        
        if self.index >= len(self.results):
            raise StopAsyncIteration
        
        item = self.results[self.index]
        self.index += 1
        return item

async def main():
    async for row in AsyncCursor("SELECT * FROM users"):
        print(row)
```

### aiter() and anext() Built-ins

```python
# Python 3.10+
async def main():
    ait = aiter(async_generator())
    
    # Get first item
    first = await anext(ait)
    
    # With default
    item = await anext(ait, "default")
```

### Combining Async Iterators

```python
async def merge_streams(*streams):
    """Merge multiple async iterators."""
    pending = set()
    iterators = {aiter(s): s for s in streams}
    
    for it in iterators:
        pending.add(asyncio.create_task(anext(it, StopAsyncIteration)))
    
    while pending:
        done, pending = await asyncio.wait(
            pending, return_when=asyncio.FIRST_COMPLETED
        )
        for task in done:
            result = task.result()
            if result is not StopAsyncIteration:
                yield result
```

## Common Pitfalls
1. **Making `__aiter__` async** — `__aiter__` must be a regular method (not `async def`) that returns the iterator object. Making it async causes `async for` to fail with a `TypeError`.
2. **Forgetting `StopAsyncIteration`** — If `__anext__` never raises `StopAsyncIteration`, the `async for` loop runs forever. Always ensure there is a termination condition.
3. **Reusing an exhausted iterator** — Once an async iterator raises `StopAsyncIteration`, it is spent. Create a new instance to iterate again. Consider implementing `__aiter__` to return a fresh iterator if reuse is needed.

## Best Practices
1. **Use async generators for simple cases** — If your async iterator is just yielding values from an async source, an async generator is simpler and less error-prone than a full class implementation.
2. **Use `anext()` with a default for safe single-item consumption** — `await anext(ait, None)` returns `None` if the iterator is exhausted instead of raising an exception.

## Summary
- Async iterators implement `__aiter__()` and `__anext__()` to support `async for` loops.
- `StopAsyncIteration` signals the end of the async sequence.
- `aiter()` and `anext()` (Python 3.10+) provide convenient built-in access to async iterators.
- Multiple async streams can be merged by scheduling `anext()` tasks and processing results as they complete.
- Prefer async generators for simple use cases; use the full protocol for complex stateful iterators.

## Code Examples

**Async file reader**

```python
class AsyncFileReader:
    """Read file lines asynchronously."""
    
    def __init__(self, filename, chunk_size=8192):
        self.filename = filename
        self.chunk_size = chunk_size
        self.file = None
        self.buffer = ""
    
    def __aiter__(self):
        return self
    
    async def __anext__(self):
        while "\n" not in self.buffer:
            if self.file is None:
                self.file = await aiofiles.open(self.filename)
            
            chunk = await self.file.read(self.chunk_size)
            if not chunk:
                if self.buffer:
                    line = self.buffer
                    self.buffer = ""
                    return line
                await self.file.close()
                raise StopAsyncIteration
            
            self.buffer += chunk
        
        line, self.buffer = self.buffer.split("\n", 1)
        return line
```


## Resources

- [Async Iterators](https://docs.python.org/3.14/reference/datamodel.html#async-iterators) — The __aiter__ and __anext__ protocol for asynchronous iteration

---

> 📘 *This lesson is part of the [Python Concurrency: Asyncio & Free-Threading](https://stanza.dev/courses/python-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*