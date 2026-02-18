---
source_course: "python-concurrency"
source_lesson: "python-concurrency-async-iterators"
---

# Async Iterators

Async iterators implement `__aiter__` and `__anext__` for `async for` loops.

## The Protocol

```python
class AsyncIterator:
    def __aiter__(self):
        return self
    
    async def __anext__(self):
        # Return next value or raise StopAsyncIteration
        pass
```

## Example: Database Cursor

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

## aiter() and anext() Built-ins

```python
# Python 3.10+
async def main():
    ait = aiter(async_generator())
    
    # Get first item
    first = await anext(ait)
    
    # With default
    item = await anext(ait, "default")
```

## Combining Async Iterators

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


---

> 📘 *This lesson is part of the [Python Concurrency: Asyncio & No-GIL](https://stanza.dev/courses/python-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*