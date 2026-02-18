---
source_course: "python-concurrency"
source_lesson: "python-concurrency-real-world-async"
---

# Production Async Patterns

## Connection Pooling

```python
class ConnectionPool:
    def __init__(self, max_connections=10):
        self.semaphore = asyncio.Semaphore(max_connections)
        self.pool = asyncio.Queue(maxsize=max_connections)
    
    @asynccontextmanager
    async def acquire(self):
        async with self.semaphore:
            try:
                conn = self.pool.get_nowait()
            except asyncio.QueueEmpty:
                conn = await create_connection()
            
            try:
                yield conn
            finally:
                await self.pool.put(conn)
```

## Rate Limiting

```python
class RateLimiter:
    def __init__(self, rate, per_seconds):
        self.rate = rate
        self.per_seconds = per_seconds
        self.tokens = rate
        self.last_update = time.monotonic()
        self.lock = asyncio.Lock()
    
    async def acquire(self):
        async with self.lock:
            now = time.monotonic()
            elapsed = now - self.last_update
            self.tokens = min(
                self.rate,
                self.tokens + elapsed * (self.rate / self.per_seconds)
            )
            self.last_update = now
            
            if self.tokens < 1:
                sleep_time = (1 - self.tokens) / (self.rate / self.per_seconds)
                await asyncio.sleep(sleep_time)
                self.tokens = 0
            else:
                self.tokens -= 1
```

## Graceful Shutdown

```python
async def main():
    tasks = set()
    shutdown = asyncio.Event()
    
    def handle_signal():
        shutdown.set()
    
    loop = asyncio.get_event_loop()
    loop.add_signal_handler(signal.SIGTERM, handle_signal)
    
    async with asyncio.TaskGroup() as tg:
        while not shutdown.is_set():
            task = tg.create_task(worker())
            tasks.add(task)
            task.add_done_callback(tasks.discard)
    
    # All tasks complete or cancelled here
```

## Circuit Breaker

```python
class CircuitBreaker:
    def __init__(self, failure_threshold=5, reset_timeout=30):
        self.failures = 0
        self.threshold = failure_threshold
        self.reset_timeout = reset_timeout
        self.last_failure = None
        self.state = "closed"  # closed, open, half-open
    
    async def call(self, func, *args):
        if self.state == "open":
            if time.time() - self.last_failure > self.reset_timeout:
                self.state = "half-open"
            else:
                raise CircuitOpenError()
        
        try:
            result = await func(*args)
            self.failures = 0
            self.state = "closed"
            return result
        except Exception:
            self.failures += 1
            self.last_failure = time.time()
            if self.failures >= self.threshold:
                self.state = "open"
            raise
```

## Code Examples

**Production-ready API client**

```python
# Combining patterns: Rate-limited, pooled API client
class APIClient:
    def __init__(self):
        self.pool = ConnectionPool(max_connections=10)
        self.rate_limiter = RateLimiter(rate=100, per_seconds=60)
        self.circuit = CircuitBreaker()
    
    async def request(self, endpoint):
        await self.rate_limiter.acquire()
        
        async with self.pool.acquire() as conn:
            return await self.circuit.call(
                conn.request, endpoint
            )

async def main():
    client = APIClient()
    
    async with asyncio.TaskGroup() as tg:
        for i in range(1000):
            tg.create_task(client.request(f"/api/item/{i}"))
```


---

> 📘 *This lesson is part of the [Python Concurrency: Asyncio & No-GIL](https://stanza.dev/courses/python-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*