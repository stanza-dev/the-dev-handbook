---
source_course: "redis-caching"
source_lesson: "redis-caching-request-coalescing"
---

# Request Coalescing and Deduplication

When a cache miss occurs, multiple simultaneous requests can overwhelm your database. Request coalescing ensures only one request fetches the data.

## The Stampede Problem

```
┌─────────────────────────────────────────────────────────────┐
│  Cache expires at T=0                                       │
│                                                             │
│  T=0.001: Request 1 → MISS → Query DB                      │
│  T=0.002: Request 2 → MISS → Query DB (duplicate!)         │
│  T=0.003: Request 3 → MISS → Query DB (duplicate!)         │
│                                                             │
│  Result: 3 identical DB queries instead of 1               │
└─────────────────────────────────────────────────────────────┘
```

## Solution 1: Distributed Lock

Only one process fetches; others wait:

```redis
# Request 1 tries to acquire lock
SET cache:user:1001:lock "req1" NX EX 10
# Returns OK - lock acquired

# Request 2 tries to acquire lock
SET cache:user:1001:lock "req2" NX EX 10
# Returns nil - already locked

# Request 1: Fetch from DB, update cache, delete lock
SET cache:user:1001 "{...}" EX 300
DEL cache:user:1001:lock

# Request 2: Retry GET, now finds cached data
GET cache:user:1001
```

### Implementation Pattern

```python
import time
import redis

def get_with_coalescing(r, key, fetch_func, ttl=300, lock_ttl=10):
    # Try cache first
    data = r.get(key)
    if data:
        return data
    
    lock_key = f"{key}:lock"
    
    # Try to acquire lock
    if r.set(lock_key, "1", nx=True, ex=lock_ttl):
        try:
            # We got the lock - fetch data
            data = fetch_func()
            r.set(key, data, ex=ttl)
            return data
        finally:
            r.delete(lock_key)
    else:
        # Someone else is fetching - wait and retry
        for _ in range(50):  # Wait up to 5 seconds
            time.sleep(0.1)
            data = r.get(key)
            if data:
                return data
        
        # Timeout - fetch ourselves
        return fetch_func()
```

## Solution 2: Probabilistic Early Refresh

Refresh cache before it expires:

```python
import random
import time

def get_with_early_refresh(r, key, fetch_func, ttl=300):
    data_json = r.get(key)
    
    if data_json:
        data = json.loads(data_json)
        remaining_ttl = r.ttl(key)
        
        # Probabilistic refresh in last 10% of TTL
        if remaining_ttl < ttl * 0.1:
            refresh_probability = 1 - (remaining_ttl / (ttl * 0.1))
            if random.random() < refresh_probability:
                # Refresh in background
                refresh_async(key, fetch_func, ttl)
        
        return data['value']
    
    # Cache miss
    value = fetch_func()
    r.set(key, json.dumps({'value': value}), ex=ttl)
    return value
```

## Solution 3: Singleflight Pattern

Application-level deduplication (no Redis lock needed):

```python
import threading
from concurrent.futures import Future

class Singleflight:
    def __init__(self):
        self._lock = threading.Lock()
        self._calls = {}  # key -> Future
    
    def do(self, key, func):
        with self._lock:
            if key in self._calls:
                # Someone already fetching - wait for their result
                return self._calls[key].result()
            
            future = Future()
            self._calls[key] = future
        
        try:
            result = func()
            future.set_result(result)
            return result
        except Exception as e:
            future.set_exception(e)
            raise
        finally:
            with self._lock:
                del self._calls[key]

# Usage
sf = Singleflight()
data = sf.do("user:1001", lambda: fetch_from_db(1001))
```

## When to Use Each

| Pattern | Best For |
|---------|----------|
| Distributed Lock | Multi-server deployments |
| Early Refresh | Predictable access patterns |
| Singleflight | Single-server, high concurrency |

📖 [Distributed Locks](https://redis.io/docs/latest/develop/use/patterns/distributed-locks/)

---

> 📘 *This lesson is part of the [High-Performance Caching Strategies](https://stanza.dev/courses/redis-caching) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*