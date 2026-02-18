---
source_course: "python-concurrency"
source_lesson: "python-concurrency-async-generators"
---

# Async Generators

Async generators combine `async def` with `yield` for streaming async data.

## Basic Async Generator

```python
async def async_range(n):
    for i in range(n):
        await asyncio.sleep(0.1)  # Async operation
        yield i

async def main():
    async for num in async_range(5):
        print(num)
```

## Paginated API Example

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

## Async Comprehensions

```python
# Async list comprehension
results = [item async for item in async_generator()]

# With condition
filtered = [x async for x in source if x > 0]

# Async generator expression
gen = (x * 2 async for x in async_range(10))
```

## Handling Cleanup

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

## Code Examples

**Ticker async generator**

```python
import asyncio

async def ticker(interval, count):
    """Yield timestamps at regular intervals."""
    for _ in range(count):
        await asyncio.sleep(interval)
        yield asyncio.get_event_loop().time()

async def main():
    async for timestamp in ticker(0.5, 5):
        print(f"Tick at {timestamp:.2f}")

asyncio.run(main())
```


---

> 📘 *This lesson is part of the [Python Concurrency: Asyncio & No-GIL](https://stanza.dev/courses/python-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*