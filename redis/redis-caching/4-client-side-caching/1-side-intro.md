---
source_course: "redis-caching"
source_lesson: "redis-caching-client-side-intro"
---

# Client-Side Caching Introduction

Client-side caching stores data in your application's memory, eliminating network round-trips to Redis entirely. Redis 7.4+ provides server-assisted invalidation to keep client caches in sync.

## Why Client-Side Caching?

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

📖 [Client-Side Caching](https://redis.io/docs/latest/develop/use/client-side-caching/)

## Resources

- [Client-Side Caching](https://redis.io/docs/latest/develop/use/client-side-caching/) — Redis server-assisted client-side caching

---

> 📘 *This lesson is part of the [High-Performance Caching Strategies](https://stanza.dev/courses/redis-caching) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*