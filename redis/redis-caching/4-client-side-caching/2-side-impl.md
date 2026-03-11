---
source_course: "redis-caching"
source_lesson: "redis-caching-client-side-impl"
---

# Implementing Client-Side Caching

## Introduction
Most modern Redis client libraries provide built-in support for client-side caching. Enabling it typically requires a RESP3 connection and a one-line configuration change.

## Key Concepts
- **RESP3 Connection**: The protocol version that supports push messages for invalidation.
- **Cache Configuration**: Client libraries expose options for local TTL, max entries, and eviction policy.
- **Transparent Caching**: Once enabled, GET/HGET commands are automatically served from local cache on repeat reads.

## Real World Context
Adding client-side caching to an existing application often requires only a configuration change — no code logic changes needed. This makes it one of the highest-ROI optimizations available for read-heavy services.

## Deep Dive

### Python (redis-py)

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

## Common Pitfalls
1. **Forgetting RESP3** — Client-side caching silently does nothing without RESP3. Always verify with CLIENT TRACKINGINFO.
2. **Assuming all commands are cached** — Only read commands (GET, HGET, MGET, etc.) use the local cache. Write commands always go to the server.

## Best Practices
1. **Verify caching is active with MONITOR** — If the same GET appears twice in MONITOR output, client-side caching is not working.
2. **Set appropriate local TTL** — Even with server invalidation, a local TTL provides defense against missed invalidation messages.

## Summary
- Enable client-side caching with protocol=3 and a cache config in your client library
- Python, Node.js, and Java all have built-in support in recent versions
- Verify with redis-cli MONITOR: cached reads should not appear on repeated access
- Only read commands (GET, HGET, MGET) are cached locally

📖 [Client-Side Caching Reference](https://redis.io/docs/latest/develop/reference/client-side-caching/)

## Code Examples

**Node.js client-side caching with node-redis — RESP3 connection with LRU local cache**

```javascript
import { createClient } from 'redis';

const client = createClient({
  socket: { host: 'localhost', port: 6379 },
  RESP: 3,  // RESP3 required for tracking
  clientSideCache: {
    ttl: 0,              // 0 = no local expiration
    maxEntries: 10000,   // limit local cache size
    evictPolicy: 'LRU'   // local eviction: LRU or FIFO
  }
});

await client.connect();
const val1 = await client.get('key'); // from Redis
const val2 = await client.get('key'); // from local cache!
```


## Resources

- [Client-Side Caching Reference](https://redis.io/docs/latest/develop/reference/client-side-caching/) — Technical details of client-side caching

---

> 📘 *This lesson is part of the [High-Performance Caching Strategies](https://stanza.dev/courses/redis-caching) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*