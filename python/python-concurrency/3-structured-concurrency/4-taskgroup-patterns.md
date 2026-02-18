---
source_course: "python-concurrency"
source_lesson: "python-concurrency-taskgroup-patterns"
---

# Common TaskGroup Patterns

## Batching with TaskGroup

```python
async def process_batch(items, batch_size=10):
    results = []
    
    for i in range(0, len(items), batch_size):
        batch = items[i:i + batch_size]
        
        async with asyncio.TaskGroup() as tg:
            tasks = [tg.create_task(process(item)) for item in batch]
        
        results.extend(task.result() for task in tasks)
    
    return results
```

## Timeout with TaskGroup

```python
async def with_timeout():
    try:
        async with asyncio.timeout(5.0):
            async with asyncio.TaskGroup() as tg:
                tg.create_task(slow_operation())
                tg.create_task(another_operation())
    except TimeoutError:
        print("Operations timed out")
        # All tasks automatically cancelled
```

## Semaphore-Limited TaskGroup

```python
async def limited_concurrent(urls, max_concurrent=5):
    semaphore = asyncio.Semaphore(max_concurrent)
    
    async def fetch_limited(url):
        async with semaphore:
            return await fetch(url)
    
    async with asyncio.TaskGroup() as tg:
        tasks = [tg.create_task(fetch_limited(url)) for url in urls]
    
    return [task.result() for task in tasks]
```

## Producer-Consumer with TaskGroup

```python
async def producer_consumer():
    queue = asyncio.Queue(maxsize=100)
    
    async def producer():
        for i in range(1000):
            await queue.put(i)
        # Signal completion
        for _ in range(num_consumers):
            await queue.put(None)
    
    async def consumer():
        while True:
            item = await queue.get()
            if item is None:
                break
            await process(item)
    
    num_consumers = 5
    async with asyncio.TaskGroup() as tg:
        tg.create_task(producer())
        for _ in range(num_consumers):
            tg.create_task(consumer())
```

## Code Examples

**Fallback pattern**

```python
# Retry pattern with TaskGroup
async def fetch_with_fallbacks(primary_url, fallback_urls):
    """Try primary, fall back to alternates."""
    async def try_fetch(url):
        result = await fetch(url)
        if result:
            return result
        raise FetchError(url)
    
    try:
        async with asyncio.TaskGroup() as tg:
            task = tg.create_task(try_fetch(primary_url))
        return task.result()
    except* FetchError:
        # Primary failed, try fallbacks
        async with asyncio.TaskGroup() as tg:
            tasks = [tg.create_task(try_fetch(url)) for url in fallback_urls]
        
        for task in tasks:
            if not task.cancelled() and task.exception() is None:
                return task.result()
        raise AllFetchesFailed()
```


---

> 📘 *This lesson is part of the [Python Concurrency: Asyncio & No-GIL](https://stanza.dev/courses/python-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*