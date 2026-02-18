---
source_course: "redis-caching"
source_lesson: "redis-caching-pipelining"
---

# Pipelining for Batch Operations

Pipelining dramatically improves performance when executing multiple Redis commands by reducing network round-trips.

## The Network Latency Problem

```
┌─────────────────────────────────────────────────────────────┐
│  Without Pipelining (3 commands = 3 round-trips):          │
│                                                             │
│  Client → GET key1 → Server → Response (~1ms)              │
│  Client → GET key2 → Server → Response (~1ms)              │
│  Client → GET key3 → Server → Response (~1ms)              │
│  Total: ~3ms                                               │
├─────────────────────────────────────────────────────────────┤
│  With Pipelining (3 commands = 1 round-trip):              │
│                                                             │
│  Client → [GET key1, GET key2, GET key3] → Server          │
│  Client ← [response1, response2, response3] ← Server       │
│  Total: ~1ms (3x faster!)                                  │
└─────────────────────────────────────────────────────────────┘
```

## Implementation

### Python Example

```python
import redis
import time

r = redis.Redis()

# Without pipeline - 1000 round-trips
start = time.time()
for i in range(1000):
    r.set(f'key:{i}', f'value:{i}')
print(f"Without pipeline: {time.time() - start:.3f}s")
# ~1.5 seconds

# With pipeline - 1 round-trip
start = time.time()
pipe = r.pipeline(transaction=False)
for i in range(1000):
    pipe.set(f'key:{i}', f'value:{i}')
pipe.execute()
print(f"With pipeline: {time.time() - start:.3f}s")
# ~0.015 seconds (100x faster!)
```

### Batch Size Optimization

```python
def bulk_operation(keys, batch_size=1000):
    """Process keys in optimal batch sizes"""
    for i in range(0, len(keys), batch_size):
        batch = keys[i:i + batch_size]
        pipe = r.pipeline(transaction=False)
        
        for key in batch:
            pipe.get(key)
        
        results = pipe.execute()
        yield from zip(batch, results)
```

## Pipeline vs Transaction

```python
# Pipeline only (no atomicity, best performance)
pipe = r.pipeline(transaction=False)

# Pipeline with transaction (MULTI/EXEC wrapper)
pipe = r.pipeline()  # transaction=True by default
```

| Mode | Atomicity | Performance | Use Case |
|------|-----------|-------------|----------|
| transaction=False | No | Best | Independent reads/writes |
| transaction=True | Yes | Good | Related operations |

## Cache Warming with Pipelining

```python
def warm_cache(keys_to_warm, fetch_func):
    """Pre-populate cache efficiently"""
    # Check which keys are missing
    pipe = r.pipeline(transaction=False)
    for key in keys_to_warm:
        pipe.exists(key)
    
    exists = pipe.execute()
    missing = [k for k, e in zip(keys_to_warm, exists) if not e]
    
    if not missing:
        return
    
    # Fetch data for missing keys
    data = fetch_func(missing)
    
    # Populate cache in batch
    pipe = r.pipeline(transaction=False)
    for key, value in zip(missing, data):
        pipe.set(key, value, ex=300)
    pipe.execute()
    
    print(f"Warmed {len(missing)} cache entries")
```

📖 [Redis Pipelining](https://redis.io/docs/latest/develop/use/pipelining/)

## Resources

- [Redis Pipelining](https://redis.io/docs/latest/develop/use/pipelining/) — Guide to Redis pipelining

---

> 📘 *This lesson is part of the [High-Performance Caching Strategies](https://stanza.dev/courses/redis-caching) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*