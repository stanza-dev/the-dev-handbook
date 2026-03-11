---
source_course: "redis-caching"
source_lesson: "redis-caching-pipelining"
---

# Pipelining for Batch Operations

## Introduction
Pipelining dramatically improves performance when executing multiple Redis commands by reducing network round-trips. Instead of waiting for each response, you send all commands at once.

## Key Concepts
- **Pipeline**: A client-side buffer that accumulates commands and sends them in a single network write.
- **Round-Trip Time (RTT)**: The latency of sending a command and receiving its response, typically 0.1-1ms on a local network.
- **Batch Size**: The number of commands grouped into a single pipeline execution.

## Real World Context
Cache warming, bulk invalidation, and multi-key reads all involve hundreds or thousands of Redis commands. Without pipelining, each command waits for the previous response, turning a 15ms operation into 1.5 seconds.

## Deep Dive

### The Network Latency Problem

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

## Common Pitfalls
1. **Pipelines that are too large** — Sending 1 million commands in a single pipeline can exhaust client memory. Batch in groups of 1,000-10,000.
2. **Using transactions when not needed** — `pipeline(transaction=True)` wraps commands in MULTI/EXEC, adding overhead. Use `transaction=False` for independent operations.

## Best Practices
1. **Use pipelines for any multi-command operation** — Even 3-5 commands benefit from pipelining when latency matters.
2. **Batch in reasonable sizes** — 1,000-5,000 commands per pipeline execution is typically optimal.

## Summary
- Pipelining reduces N round-trips to 1, providing 10-100x speedup for batch operations
- Use `transaction=False` for independent commands (better performance)
- Optimal batch size is 1,000-5,000 commands per execute call
- Essential for cache warming, bulk invalidation, and multi-key reads

📖 [Redis Pipelining](https://redis.io/docs/latest/develop/use/pipelining/)

## Code Examples

**Pipeline vs individual commands — 100x speedup by batching 1000 SET commands into a single network round-trip**

```python
import redis

r = redis.Redis()

# Without pipeline: 1000 round-trips
for i in range(1000):
    r.set(f'key:{i}', f'value:{i}')  # ~1.5 seconds

# With pipeline: 1 round-trip for 1000 commands
pipe = r.pipeline(transaction=False)
for i in range(1000):
    pipe.set(f'key:{i}', f'value:{i}')
pipe.execute()  # ~0.015 seconds (100x faster!)
```


## Resources

- [Redis Pipelining](https://redis.io/docs/latest/develop/use/pipelining/) — Guide to Redis pipelining

---

> 📘 *This lesson is part of the [High-Performance Caching Strategies](https://stanza.dev/courses/redis-caching) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*