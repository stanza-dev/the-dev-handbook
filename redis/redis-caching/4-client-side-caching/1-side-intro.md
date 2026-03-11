---
source_course: "redis-caching"
source_lesson: "redis-caching-client-side-intro"
---

# Client-Side Caching Introduction

## Introduction
Client-side caching stores data in your application's memory, eliminating network round-trips to Redis entirely. Redis provides server-assisted invalidation to keep client caches in sync automatically.

## Key Concepts
- **Client-Side Cache**: A local in-memory store within your application process.
- **Tracking**: Redis server feature that remembers which keys each client has read and sends invalidation messages when those keys change.
- **Invalidation Message**: A push notification from server to client indicating a cached key has been modified.

## Real World Context
For latency-critical applications like real-time dashboards or gaming leaderboards, even the 1ms network round-trip to Redis matters. Client-side caching eliminates this entirely for frequently read data, achieving sub-microsecond reads from local memory.

## Deep Dive

### Why Client-Side Caching?

```
┌─────────────────────────────────────────────────────────────┐
│  Without Client-Side Cache:                                  │
│                                                             │
│  App → Network (0.5ms) → Redis (0.1ms) → Network → App      │
│  Total: ~1ms per request                                    │
├─────────────────────────────────────────────────────────────┤
│  With Client-Side Cache:                                     │
│                                                             │
│  App → Local Memory (0.001ms) → App                         │
│  Total: ~0.001ms per request (1000x faster!)               │
└─────────────────────────────────────────────────────────────┘
```

**Benefits:**
- Near-zero latency for cached data
- No network traffic for hits
- Reduced load on Redis server

## The Challenge: Stale Data

The problem with local caching is invalidation:

```
┌─────────────────────────────────────────────────────────────┐
│  T=0: Client A caches user:1001 = {"name": "Alice"}        │
│  T=1: Client B updates user:1001 = {"name": "Alicia"}      │
│  T=2: Client A reads user:1001 → Returns stale "Alice"!    │
└─────────────────────────────────────────────────────────────┘
```

## Redis Tracking Mode

Redis solves this with **tracking**. The server remembers which keys each client has read and sends invalidation messages when those keys change.

```
┌─────────────────────────────────────────────────────────────┐
│  1. Client A: GET user:1001                                 │
│  2. Redis: Records "Client A is tracking user:1001"        │
│  3. Client A: Caches locally                               │
│                                                             │
│  4. Client B: SET user:1001 "new value"                    │
│  5. Redis: Sends invalidation to Client A                  │
│  6. Client A: Evicts user:1001 from local cache            │
│                                                             │
│  7. Client A: GET user:1001 → Fetches fresh from Redis     │
└─────────────────────────────────────────────────────────────┘
```

## Requirements

- **Redis 7.4+** (or Redis 6.0+ with limited features)
- **RESP3 protocol** (required for tracking)
- **Client library support** (most modern libraries support this)

## Tracking Modes

### Default Mode

- Server tracks which keys each client reads
- Sends targeted invalidations
- Uses memory on server to track

### Broadcasting Mode

- Server broadcasts all invalidations
- Clients filter what they care about
- Lower server memory, more network traffic

```redis
# Enable tracking (RESP3 connection required)
CLIENT TRACKING ON

# Or with broadcasting for specific prefixes
CLIENT TRACKING ON BCAST PREFIX cache:
```

## When to Use Client-Side Caching

✅ **Good candidates:**
- Read-heavy workloads
- Low-latency requirements
- Data read repeatedly in short time
- Configuration/settings

❌ **Avoid for:**
- Frequently changing data
- One-time reads
- Large objects (consumes app memory)

## Common Pitfalls
1. **Caching large objects locally** — Client-side cache uses application heap memory. Caching large JSON blobs can cause memory pressure and garbage collection pauses.
2. **Not limiting local cache size** — Without a max entries limit, the local cache grows unbounded during traffic spikes.

## Best Practices
1. **Set a max entry count** — Limit the local cache to 10,000-50,000 entries depending on your memory budget.
2. **Target read-heavy, stable data** — Configuration settings, feature flags, and user sessions are ideal candidates for client-side caching.

## Summary
- Client-side caching eliminates network round-trips for ~1000x speed improvement
- Redis tracking sends invalidation messages when cached keys change
- RESP3 protocol is required for server-assisted invalidation
- Best suited for read-heavy workloads with stable data

📖 [Client-Side Caching](https://redis.io/docs/latest/develop/use/client-side-caching/)

## Code Examples

**Enabling Redis client-side caching tracking — default mode for targeted invalidation, BCAST for prefix-based**

```bash
# Enable client tracking (requires RESP3 connection)
CLIENT TRACKING ON

# When another client modifies a tracked key,
# the server sends an invalidation message:
# > invalidate: ["user:1001"]

# Enable broadcasting mode with prefix filter
CLIENT TRACKING ON BCAST PREFIX cache:
```


## Resources

- [Client-Side Caching](https://redis.io/docs/latest/develop/use/client-side-caching/) — Redis server-assisted client-side caching

---

> 📘 *This lesson is part of the [High-Performance Caching Strategies](https://stanza.dev/courses/redis-caching) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*