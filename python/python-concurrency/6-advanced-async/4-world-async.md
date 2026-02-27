---
source_course: "python-concurrency"
source_lesson: "python-concurrency-real-world-async"
---

# Real-World Async Patterns

## Introduction
Production async applications face challenges beyond basic concurrency: managing connection pools, enforcing rate limits, handling graceful shutdowns, and protecting against cascading failures. This lesson presents four battle-tested patterns that form the backbone of resilient async services: connection pooling, rate limiting, graceful shutdown, and circuit breakers.

## Key Concepts
- **Connection pool**: A fixed set of reusable connections managed by a semaphore and queue, avoiding the overhead of creating a new connection per request.
- **Rate limiter**: A token bucket algorithm that controls how many operations are allowed per time period.
- **Graceful shutdown**: A pattern where signal handlers set an event to stop accepting new work while allowing in-flight tasks to complete.
- **Circuit breaker**: A pattern that stops calling a failing service after a threshold of errors, allowing it time to recover before retrying.

## Real World Context
A microservices backend calls three external APIs, a database, and a cache. Each external dependency can slow down, fail, or rate-limit you. A connection pool reuses database connections efficiently. A rate limiter prevents hitting API quotas. A circuit breaker stops hammering a failing service so it can recover. Graceful shutdown ensures all in-flight requests complete before the process exits during a deploy.

## Deep Dive

### Connection Pooling

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

### Rate Limiting

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

### Graceful Shutdown

```python
async def main():
    tasks = set()
    shutdown = asyncio.Event()
    
    def handle_signal():
        shutdown.set()
    
    loop = asyncio.get_running_loop()
    loop.add_signal_handler(signal.SIGTERM, handle_signal)
    
    async with asyncio.TaskGroup() as tg:
        while not shutdown.is_set():
            task = tg.create_task(worker())
            tasks.add(task)
            task.add_done_callback(tasks.discard)
    
    # All tasks complete or cancelled here
```

### Circuit Breaker

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

## Common Pitfalls
1. **Not returning connections to the pool on error** — If an exception occurs while using a pooled connection and the `finally` block is missing, the connection is lost forever, eventually exhausting the pool.
2. **Setting circuit breaker thresholds too low** — A threshold of 1-2 failures causes the circuit to open on transient errors. Use a threshold that distinguishes transient blips from sustained outages (typically 5-10).
3. **Not handling SIGTERM in async services** — Container orchestrators like Kubernetes send SIGTERM before killing a process. Without a handler, in-flight requests are aborted mid-execution.

## Best Practices
1. **Compose patterns into a single client class** — Combine connection pooling, rate limiting, and circuit breaking into a unified API client that handles all resilience concerns transparently.
2. **Use `asyncio.Event` for shutdown coordination** — It integrates cleanly with async code and can be checked in loops, used with `asyncio.wait()`, or awaited directly.
3. **Monitor your circuit breaker state** — Log state transitions (closed -> open -> half-open) so you can correlate them with downstream service incidents.

## Summary
- Connection pools reuse connections via a semaphore and queue, reducing connection setup overhead.
- Token-bucket rate limiters control throughput to stay within API quotas.
- Graceful shutdown uses signal handlers and `asyncio.Event` to drain in-flight work before exiting.
- Circuit breakers protect against cascading failures by stopping calls to failing services.
- These patterns compose together to build resilient, production-grade async services.

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


## Resources

- [asyncio Module](https://docs.python.org/3.14/library/asyncio.html) — Comprehensive asyncio reference for production async patterns

---

> 📘 *This lesson is part of the [Python Concurrency: Asyncio & Free-Threading](https://stanza.dev/courses/python-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*