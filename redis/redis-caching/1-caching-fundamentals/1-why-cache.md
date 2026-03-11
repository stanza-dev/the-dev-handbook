---
source_course: "redis-caching"
source_lesson: "redis-caching-why-cache"
---

# Why Caching Matters

## Introduction
Caching is one of the most effective ways to improve application performance. By storing frequently accessed data in Redis, you can reduce database load and dramatically speed up response times.

## Key Concepts
- **Cache Hit**: When requested data is found in the cache, avoiding a database query.
- **Cache Miss**: When data is not in the cache, requiring a database lookup.
- **Hit Rate**: The percentage of requests served from cache — aim for 95%+ for high-traffic applications.
- **Staleness**: When cached data no longer matches the source of truth in the database.

## Real World Context
Every high-traffic website relies on caching to stay responsive. Without it, each page view triggers expensive database queries. Redis handles millions of operations per second from RAM, making it the most popular caching solution.

## Deep Dive

### The Performance Problem

Without caching, every request hits your database:

```
┌──────────────────────────────────────────────────────────────┐
│  Without Cache:                                               │
│                                                              │
│  User Request → App Server → Database (100ms) → Response     │
│  User Request → App Server → Database (100ms) → Response     │
│  User Request → App Server → Database (100ms) → Response     │
│                                                              │
│  100 users = 100 database queries = HIGH LOAD                │
└──────────────────────────────────────────────────────────────┘
```

With caching, subsequent requests are instant:

```
┌──────────────────────────────────────────────────────────────┐
│  With Cache:                                                  │
│                                                              │
│  User Request → App Server → Cache MISS → DB → Response      │
│  User Request → App Server → Cache HIT (1ms) → Response      │
│  User Request → App Server → Cache HIT (1ms) → Response      │
│                                                              │
│  100 users = 1 database query + 99 cache hits = LOW LOAD     │
└──────────────────────────────────────────────────────────────┘
```

## Performance Comparison

| Operation | Database | Redis Cache |
|-----------|----------|-------------|
| Simple query | 10-100ms | 0.1-1ms |
| Complex query | 100-1000ms | 0.1-1ms |
| Network round-trip | 1-10ms | 0.1-1ms |

**Redis is 10-1000x faster than typical database queries.**

## What to Cache

### Good Candidates for Caching

✅ **Expensive computations**
- Complex database queries with joins
- Aggregation results
- Search results

✅ **Frequently accessed data**
- User profiles
- Product listings
- Configuration settings

✅ **Slowly changing data**
- Category lists
- Geographic data
- Exchange rates

✅ **API responses**
- Third-party API results
- Computed API responses

### Poor Candidates

❌ **Rapidly changing data** (real-time stock prices)
❌ **Highly personalized data** (unique per user, low reuse)
❌ **Data that must always be current** (bank balances during transactions)
❌ **Very large objects** (videos, large files)

## Cache Hit Rate

The cache hit rate measures caching effectiveness:

```
Hit Rate = Cache Hits / (Cache Hits + Cache Misses) × 100%
```

| Hit Rate | Assessment |
|----------|------------|
| 95%+ | Excellent |
| 80-95% | Good |
| 60-80% | Needs improvement |
| <60% | Poor - review caching strategy |

## Caching Trade-offs

### Pros
- Dramatically faster response times
- Reduced database load
- Better scalability
- Protection during traffic spikes

### Cons
- Data can become stale
- Added complexity
- Memory costs
- Cache invalidation challenges

> "There are only two hard things in Computer Science: cache invalidation and naming things." - Phil Karlton

## Common Pitfalls
1. **Caching everything** — Not all data benefits from caching. Rapidly changing data or highly personalized content yields low hit rates and wastes memory.
2. **Ignoring cache hit rate** — Deploying a cache without monitoring hits vs misses means you cannot tell if it is actually helping.

## Best Practices
1. **Start with the highest-impact data** — Cache the slowest or most frequently accessed queries first for maximum benefit.
2. **Always set a TTL** — Every cache entry should expire to prevent stale data and unbounded memory growth.

## Summary
- Caching reduces database load and speeds up responses by 10-1000x
- Good cache candidates are expensive queries, frequently accessed data, and slowly changing content
- Cache hit rate is the key metric — aim for 95%+
- Every caching strategy involves trade-offs between freshness and performance

## Code Examples

**Basic cache pattern — store a database result in Redis with a TTL so future reads skip the database**

```bash
# Cache a database query result with a 5-minute TTL
SET cache:user:1001 '{"name":"Alice","email":"a@ex.com"}' EX 300

# Subsequent reads are served from cache (~0.1ms vs ~50ms from DB)
GET cache:user:1001
# Output: {"name":"Alice","email":"a@ex.com"}
```


## Resources

- [Redis as a Cache](https://redis.io/docs/latest/develop/) — Official Redis development documentation and use cases

---

> 📘 *This lesson is part of the [High-Performance Caching Strategies](https://stanza.dev/courses/redis-caching) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*