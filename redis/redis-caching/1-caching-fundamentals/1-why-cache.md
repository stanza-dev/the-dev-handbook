---
source_course: "redis-caching"
source_lesson: "redis-caching-why-cache"
---

# Why Caching Matters

Caching is one of the most effective ways to improve application performance. By storing frequently accessed data in Redis, you can reduce database load and dramatically speed up response times.

## The Performance Problem

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

📖 [Redis Caching Overview](https://redis.io/docs/latest/develop/use/client-side-caching/)

---

> 📘 *This lesson is part of the [High-Performance Caching Strategies](https://stanza.dev/courses/redis-caching) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*