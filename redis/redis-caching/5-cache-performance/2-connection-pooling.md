---
source_course: "redis-caching"
source_lesson: "redis-caching-connection-pooling"
---

# Connection Pooling and Serialization

Optimize your Redis client for production with proper connection management and efficient data serialization.

## Connection Pooling

Creating new connections is expensive. Connection pools reuse connections.

```python
import redis

# Bad: New connection per request
def get_value_bad(key):
    r = redis.Redis()  # Creates new connection!
    return r.get(key)

# Good: Shared connection pool
pool = redis.ConnectionPool(
    host='localhost',
    port=6379,
    max_connections=50,
    socket_timeout=5,
    socket_connect_timeout=5
)

def get_value_good(key):
    r = redis.Redis(connection_pool=pool)
    return r.get(key)
```

### Pool Configuration

```python
pool = redis.ConnectionPool(
    host='localhost',
    port=6379,
    db=0,
    password='secret',
    
    # Pool settings
    max_connections=50,           # Max pool size
    
    # Timeout settings
    socket_timeout=5.0,           # Read/write timeout
    socket_connect_timeout=2.0,   # Connection timeout
    socket_keepalive=True,        # TCP keepalive
    
    # Retry settings
    retry_on_timeout=True,
    health_check_interval=30      # Periodic health checks
)
```

## Serialization Strategies

Choose the right serialization for your use case:

| Format | Size | Speed | Human Readable |
|--------|------|-------|----------------|
| JSON | Large | Slow | Yes |
| MessagePack | Small | Fast | No |
| Pickle | Medium | Fast | No (Python only) |
| Protocol Buffers | Smallest | Fastest | No |

### JSON Serialization

```python
import json

class JSONCache:
    def __init__(self, redis_client):
        self.r = redis_client
    
    def set(self, key, value, ttl=300):
        self.r.set(key, json.dumps(value), ex=ttl)
    
    def get(self, key):
        data = self.r.get(key)
        return json.loads(data) if data else None
```

### MessagePack (Faster + Smaller)

```python
import msgpack

class MsgPackCache:
    def __init__(self, redis_client):
        self.r = redis_client
    
    def set(self, key, value, ttl=300):
        self.r.set(key, msgpack.packb(value), ex=ttl)
    
    def get(self, key):
        data = self.r.get(key)
        return msgpack.unpackb(data, raw=False) if data else None
```

### Compression for Large Values

```python
import gzip
import json

def set_compressed(r, key, value, ttl=300):
    """Compress large values before caching"""
    json_bytes = json.dumps(value).encode('utf-8')
    
    # Only compress if worth it (>1KB)
    if len(json_bytes) > 1024:
        compressed = gzip.compress(json_bytes)
        r.set(f"{key}:gz", compressed, ex=ttl)
    else:
        r.set(key, json_bytes, ex=ttl)

def get_compressed(r, key):
    """Get and decompress if needed"""
    # Try compressed first
    data = r.get(f"{key}:gz")
    if data:
        return json.loads(gzip.decompress(data))
    
    data = r.get(key)
    return json.loads(data) if data else None
```

## Monitoring Cache Performance

```python
def get_cache_stats(r):
    """Get cache performance metrics"""
    info = r.info('stats')
    
    hits = info.get('keyspace_hits', 0)
    misses = info.get('keyspace_misses', 0)
    total = hits + misses
    
    return {
        'hits': hits,
        'misses': misses,
        'hit_rate': hits / total if total > 0 else 0,
        'ops_per_sec': info.get('instantaneous_ops_per_sec', 0)
    }
```

📖 [Redis Clients](https://redis.io/docs/latest/develop/clients/)

---

> 📘 *This lesson is part of the [High-Performance Caching Strategies](https://stanza.dev/courses/redis-caching) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*