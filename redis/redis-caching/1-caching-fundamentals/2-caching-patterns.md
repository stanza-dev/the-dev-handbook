---
source_course: "redis-caching"
source_lesson: "redis-caching-patterns"
---

# Core Caching Patterns

There are several patterns for integrating a cache into your application. Each has different trade-offs.

## Cache-Aside (Lazy Loading)

The most common pattern. The application manages the cache directly.

```
┌─────────────────────────────────────────────────────────────┐
│  READ PATH:                                                  │
│  1. App checks cache                                        │
│  2. HIT? Return cached data                                 │
│  3. MISS? Query database                                    │
│  4. Store result in cache                                   │
│  5. Return data                                             │
└─────────────────────────────────────────────────────────────┘
```

```redis
# Pseudocode implementation:

# Try to get from cache
GET cache:user:1001

# If cache miss:
# 1. Query database: SELECT * FROM users WHERE id = 1001
# 2. Store in cache
SET cache:user:1001 "{...json...}" EX 300

# Return the data
```

### Cache-Aside Pros & Cons

**Pros:**
- Only requested data is cached (efficient)
- Application controls what gets cached
- Cache failure doesn't break the app

**Cons:**
- First request always slow (cache miss)
- Data can become stale
- Extra code in application layer

## Write-Through

Data is written to cache and database simultaneously.

```
┌─────────────────────────────────────────────────────────────┐
│  WRITE PATH:                                                 │
│  1. App writes to database                                  │
│  2. App writes to cache (same transaction if possible)      │
│                                                             │
│  READ PATH:                                                  │
│  1. App reads from cache                                    │
│  2. Data always present (no cold start)                     │
└─────────────────────────────────────────────────────────────┘
```

```redis
# On every write:
# 1. Database: INSERT/UPDATE
# 2. Cache update
SET cache:user:1001 "{...updated data...}" EX 300
```

### Write-Through Pros & Cons

**Pros:**
- Cache always up-to-date
- No cache misses for written data
- Simpler read logic

**Cons:**
- Every write is slower (double write)
- Caches data that may never be read
- More complex write logic

## Write-Behind (Write-Back)

Writes go to cache first, then asynchronously to database.

```
┌─────────────────────────────────────────────────────────────┐
│  WRITE PATH:                                                 │
│  1. App writes to cache (fast!)                             │
│  2. Background process syncs to database                    │
│                                                             │
│  ⚠️  Risk: Data loss if cache fails before sync            │
└─────────────────────────────────────────────────────────────┘
```

### Write-Behind Pros & Cons

**Pros:**
- Extremely fast writes
- Can batch database writes
- Great for write-heavy workloads

**Cons:**
- Data loss risk on cache failure
- Complex to implement correctly
- Eventual consistency only

## Read-Through

The cache layer handles database access transparently.

```
┌─────────────────────────────────────────────────────────────┐
│  READ PATH:                                                  │
│  1. App requests data from cache                            │
│  2. Cache checks if data exists                             │
│  3. MISS? Cache fetches from database automatically         │
│  4. Cache stores and returns data                           │
└─────────────────────────────────────────────────────────────┘
```

This requires a cache layer that can query the database directly (like Redis with custom modules or proxy layers).

## Pattern Selection Guide

| Pattern | Best For |
|---------|----------|
| Cache-Aside | Most web applications, read-heavy workloads |
| Write-Through | When cache consistency is critical |
| Write-Behind | High write throughput, can tolerate eventual consistency |
| Read-Through | When you want transparent caching |

## Hybrid Approaches

In practice, many applications combine patterns:

```redis
# Cache-Aside for reads
GET cache:product:123
# If miss, fetch and cache

# Write-through for writes
# After database update:
DEL cache:product:123       # Invalidate cache
# OR
SET cache:product:123 "..." # Update cache
```

📖 [Caching Strategies](https://redis.io/docs/latest/develop/use/patterns/)

## Resources

- [Redis Patterns](https://redis.io/docs/latest/develop/use/patterns/) — Common Redis caching patterns

---

> 📘 *This lesson is part of the [High-Performance Caching Strategies](https://stanza.dev/courses/redis-caching) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*