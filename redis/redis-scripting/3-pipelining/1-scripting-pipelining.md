---
source_course: "redis-scripting"
source_lesson: "redis-scripting-pipelining"
---

# Understanding Pipelining

Pipelining allows you to send multiple commands without waiting for individual responses, dramatically reducing latency.

## The Network Latency Problem

```
┌─────────────────────────────────────────────────────────────┐
│  Without Pipelining (4 commands, 4 round-trips):           │
│                                                             │
│  Client → SET key1 → Server                                │
│  Client ← OK ← Server                    ~1ms              │
│  Client → SET key2 → Server                                │
│  Client ← OK ← Server                    ~1ms              │
│  Client → SET key3 → Server                                │
│  Client ← OK ← Server                    ~1ms              │
│  Client → SET key4 → Server                                │
│  Client ← OK ← Server                    ~1ms              │
│                                                             │
│  Total: ~4ms                                               │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│  With Pipelining (4 commands, 1 round-trip):               │
│                                                             │
│  Client → SET key1                                         │
│         → SET key2                                         │
│         → SET key3  → Server                               │
│         → SET key4                                         │
│  Client ← OK                                               │
│         ← OK                                               │
│         ← OK        ← Server             ~1ms total        │
│         ← OK                                               │
│                                                             │
│  Total: ~1ms (4x faster!)                                  │
└─────────────────────────────────────────────────────────────┘
```

## Basic Pipelining

```python
import redis

r = redis.Redis()

# Without pipeline - 1000 round-trips
for i in range(1000):
    r.set(f'key:{i}', f'value:{i}')
# Time: ~1000ms

# With pipeline - 1 round-trip
pipe = r.pipeline(transaction=False)
for i in range(1000):
    pipe.set(f'key:{i}', f'value:{i}')
results = pipe.execute()
# Time: ~10ms (100x faster!)
```

## Pipeline vs Transaction

```python
# Pipeline WITHOUT transaction (transaction=False)
# Commands may interleave with other clients
pipe = r.pipeline(transaction=False)

# Pipeline WITH transaction (default)
# Wraps commands in MULTI/EXEC
pipe = r.pipeline()  # transaction=True by default
```

**When to use each:**
- `transaction=False`: Best performance, OK if interleaving is acceptable
- `transaction=True`: When atomicity matters

## Reading Results

```python
pipe = r.pipeline(transaction=False)

pipe.set('name', 'Alice')
pipe.get('name')
pipe.incr('counter')
pipe.get('counter')

results = pipe.execute()
# results = [True, 'Alice', 1, '1']
#            SET   GET    INCR  GET
```

## Batching Large Operations

```python
def bulk_set(redis_client, data_dict, batch_size=1000):
    """Set many keys efficiently"""
    items = list(data_dict.items())
    
    for i in range(0, len(items), batch_size):
        batch = items[i:i+batch_size]
        pipe = redis_client.pipeline(transaction=False)
        
        for key, value in batch:
            pipe.set(key, value)
        
        pipe.execute()

# Set 100,000 keys efficiently
data = {f'key:{i}': f'value:{i}' for i in range(100000)}
bulk_set(r, data)
```

## Pipeline with Conditional Logic

```python
def get_or_create_users(user_ids):
    """Batch fetch users, create missing ones"""
    pipe = r.pipeline(transaction=False)
    
    # First, try to get all users
    for uid in user_ids:
        pipe.hgetall(f'user:{uid}')
    
    results = pipe.execute()
    
    # Find missing users
    missing = []
    users = []
    for uid, data in zip(user_ids, results):
        if data:
            users.append(data)
        else:
            missing.append(uid)
    
    # Create missing users
    if missing:
        pipe = r.pipeline(transaction=False)
        for uid in missing:
            pipe.hset(f'user:{uid}', mapping={'id': uid, 'created': 'now'})
        pipe.execute()
    
    return users
```

## Memory Considerations

```python
# Bad: Buffer might get too large
pipe = r.pipeline()
for i in range(1000000):
    pipe.set(f'key:{i}', 'x' * 10000)  # Large values
pipe.execute()  # Memory spike!

# Good: Batch in chunks
for i in range(0, 1000000, 10000):
    pipe = r.pipeline()
    for j in range(i, min(i+10000, 1000000)):
        pipe.set(f'key:{j}', 'x' * 10000)
    pipe.execute()  # Smaller batches
```

📖 [Redis Pipelining](https://redis.io/docs/latest/develop/use/pipelining/)

## Resources

- [Redis Pipelining](https://redis.io/docs/latest/develop/use/pipelining/) — Guide to Redis pipelining

---

> 📘 *This lesson is part of the [Scripting, Transactions, and Programmability](https://stanza.dev/courses/redis-scripting) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*