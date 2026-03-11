---
source_course: "redis-caching"
source_lesson: "redis-caching-redis-as-cache"
---

# Redis as a Cache Layer

## Introduction
Redis excels as a caching layer thanks to its in-memory architecture and built-in expiration support. Understanding the core commands and data type choices is essential before diving into advanced caching patterns.

## Key Concepts
- **SET with EX/PX**: Store a value with a time-to-live in seconds (EX) or milliseconds (PX).
- **Key Naming Convention**: A structured approach like `cache:{entity}:{id}` for organizing cached data.
- **Hash vs String**: Two primary Redis data types for caching objects, each with different trade-offs.

## Real World Context
Every production application that uses Redis for caching relies on these core commands daily. Choosing the right data type and key naming scheme up front prevents painful migrations later — a poorly named cache key makes invalidation nearly impossible at scale.

## Deep Dive

### Core Caching Commands

The most fundamental caching operation is SET with an expiration:

```redis
# Store with TTL (seconds)
SET cache:user:1001 '{"name":"Alice"}' EX 300

# Store with TTL (milliseconds)
SET cache:user:1001 '{"name":"Alice"}' PX 300000

# Only set if key does NOT exist (prevents race conditions)
SET cache:user:1001 '{"name":"Alice"}' NX EX 300

# Read, check TTL, delete
GET cache:user:1001
TTL cache:user:1001
DEL cache:user:1001
```

The NX flag prevents overwriting a cache entry that another process just populated, avoiding a race condition during concurrent cache misses.

### Strings vs Hashes for Caching

You have two main options for caching objects.

**Option A — Serialized String:** store the entire object as JSON.

```redis
SET cache:user:1001 '{"name":"Alice","email":"a@ex.com","age":30}' EX 300
```

Simple and works with any serialization format, but you must deserialize the entire object to read one field.

**Option B — Hash:** store fields individually.

```redis
HSET cache:user:1001 name "Alice" email "a@ex.com" age 30
EXPIRE cache:user:1001 300
```

Allows reading or updating individual fields with HGET/HSET and uses less memory for small objects. However, it does not support nested structures natively.

Use strings for opaque blobs you always read in full. Use hashes when you need partial reads or field-level updates.

### Key Naming Conventions

A consistent naming scheme is critical for operations and debugging:

```redis
# Pattern: {prefix}:{entity}:{identifier}
cache:user:1001
cache:product:abc-123
cache:api:weather:nyc
cache:session:abc123
```

Use colons as separators — tools like Redis Insight use them to build a visual key hierarchy.

## Common Pitfalls
1. **Forgetting to set a TTL** — Keys without expiration accumulate forever and eventually exhaust memory. Always set EX or call EXPIRE.
2. **Using KEYS in production** — The KEYS command blocks Redis while scanning the entire keyspace. Use SCAN for incremental, non-blocking iteration instead.

## Best Practices
1. **Always use EX/PX with SET for cache entries** — This prevents orphaned keys and ensures natural cache turnover.
2. **Keep key names concise but readable** — `cache:user:1001` is good; `application:myapp:database:users:user_id:1001:profile_data` wastes memory.

## Summary
- SET with EX/PX is the fundamental caching operation in Redis
- Choose strings for opaque blobs, hashes for partial field access
- Consistent key naming (`cache:entity:id`) enables easier invalidation and monitoring
- Always set a TTL on cache entries to prevent memory exhaustion

## Code Examples

**Basic Redis caching workflow — SET with TTL, GET, check TTL, and DEL to invalidate**

```bash
# Store with 5-minute TTL
SET cache:user:1001 '{"name":"Alice","email":"a@ex.com"}' EX 300

# Read the cached value
GET cache:user:1001
# Output: {"name":"Alice","email":"a@ex.com"}

# Check remaining time-to-live
TTL cache:user:1001
# Output: (integer) 297

# Invalidate the cache entry
DEL cache:user:1001
# Output: (integer) 1
```


## Resources

- [SET Command](https://redis.io/docs/latest/commands/set/) — Official documentation for the SET command including EX, PX, NX options

---

> 📘 *This lesson is part of the [High-Performance Caching Strategies](https://stanza.dev/courses/redis-caching) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*