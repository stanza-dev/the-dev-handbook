---
source_course: "redis-caching"
source_lesson: "redis-caching-client-side-impl"
---

# Implementing Client-Side Caching

Let's see how to enable client-side caching in different programming languages.

## Python (redis-py)

Requires redis-py v5.1.0+ and Redis 7.4+:

```python
import redis
from redis.cache import CacheConfig

# Create client with caching enabled
r = redis.Redis(
    host='localhost',
    port=6379,
    protocol=3,                    # RESP3 required
    cache_config=CacheConfig(),    # Enable caching
    decode_responses=True
)

# First read - fetches from Redis and caches locally
r.set("city", "New York")
city1 = r.get("city")    # Network round-trip

# Second read - served from local cache
city2 = r.get("city")    # No network! Sub-microsecond

# If another client updates "city", Redis sends invalidation
# Next read will fetch fresh data
```

### Cache Configuration Options

```python
from redis.cache import CacheConfig

cache_config = CacheConfig(
    max_size=10000,        # Maximum entries in cache
    ttl=300,               # Local cache TTL (seconds)
    eviction_policy='lru'  # Local eviction policy
)

r = redis.Redis(
    protocol=3,
    cache_config=cache_config
)
```

## Node.js (node-redis)

Requires node-redis v5.1.0+:

```javascript
import { createClient } from 'redis';

const client = createClient({
    socket: {
        host: 'localhost',
        port: 6379
    },
    RESP: 3,                        // RESP3 required
    clientSideCache: {
        ttl: 0,                     // 0 = no local expiration
        maxEntries: 0,              // 0 = unlimited
        evictPolicy: 'LRU'          // 'LRU' or 'FIFO'
    }
});

await client.connect();

// First get - from Redis
await client.set('key', 'value');
const val1 = await client.get('key');  // Network

// Second get - from local cache
const val2 = await client.get('key');  // No network!

await client.disconnect();
```

## Java (Jedis)

Requires Jedis v5.2.0+:

```java
import redis.clients.jedis.*;
import redis.clients.jedis.csc.*;

HostAndPort endpoint = new HostAndPort("localhost", 6379);

DefaultJedisClientConfig config = DefaultJedisClientConfig
    .builder()
    .protocol(RedisProtocol.RESP3)  // RESP3 required
    .build();

CacheConfig cacheConfig = CacheConfig.builder()
    .maxSize(1000)
    .build();

UnifiedJedis client = new UnifiedJedis(endpoint, config, cacheConfig);

// Use client normally - caching is transparent
client.set("key", "value");
String val1 = client.get("key");  // From Redis
String val2 = client.get("key");  // From local cache
```

## Verifying Client-Side Caching Works

Use MONITOR to verify:

```bash
# Terminal 1: Start monitoring
redis-cli MONITOR
```

```python
# Terminal 2: Run with cache disabled
r.get("key")  # Shows in MONITOR
r.get("key")  # Shows in MONITOR again

# Run with cache enabled
r.get("key")  # Shows in MONITOR
r.get("key")  # Does NOT show - served from cache!
```

## Important Considerations

### Memory Usage

Client-side cache uses application memory:

```python
# Limit cache size to prevent memory issues
cache_config = CacheConfig(
    max_size=10000,  # Max 10,000 entries
)
```

### Connection Handling

Cache is tied to connection:

```python
# If connection drops, cache is cleared
# Reconnection starts with empty cache
```

### Not All Commands Cached

Only certain read commands are cached:
- GET, MGET
- HGET, HGETALL, HMGET
- etc.

Write commands and special commands are not cached.

📖 [Client-Side Caching Reference](https://redis.io/docs/latest/develop/reference/client-side-caching/)

## Resources

- [Client-Side Caching Reference](https://redis.io/docs/latest/develop/reference/client-side-caching/) — Technical details of client-side caching

---

> 📘 *This lesson is part of the [High-Performance Caching Strategies](https://stanza.dev/courses/redis-caching) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*