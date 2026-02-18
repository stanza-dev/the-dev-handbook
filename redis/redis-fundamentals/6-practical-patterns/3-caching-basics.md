---
source_course: "redis-fundamentals"
source_lesson: "redis-fundamentals-caching-basics"
---

# Caching Patterns

Caching is Redis's most common use case. Let's explore patterns for effective caching.

## Cache-Aside (Lazy Loading)

The most common pattern - application manages the cache:

```
┌──────────────────────────────────────────────────────┐
│  1. App checks cache                                  │
│  2. Cache HIT? → Return cached data                  │
│  3. Cache MISS? → Query database                     │
│  4. Store result in cache                            │
│  5. Return data                                      │
└──────────────────────────────────────────────────────┘
```

```redis
# Check cache
GET cache:user:1001

# If nil (cache miss):
# 1. Query database
# 2. Store in cache
SET cache:user:1001 "{...json...}" EX 300

# Return data
```

### Pros and Cons

**Pros:**
- Only requested data is cached
- Resilient to cache failures
- Simple to implement

**Cons:**
- First request is always slow (cache miss)
- Data can become stale

## Write-Through Cache

Update cache when writing to database:

```
┌──────────────────────────────────────────────────────┐
│  1. App writes to database                           │
│  2. App writes to cache                              │
│  (Both writes in same transaction if possible)       │
└──────────────────────────────────────────────────────┘
```

```redis
# After database UPDATE:
SET cache:user:1001 "{...updated json...}" EX 300

# Or delete to force refresh on next read:
DEL cache:user:1001
```

## Cache Invalidation Strategies

### Time-Based (TTL)

```redis
# Data expires after 5 minutes
SET cache:api:products "[...]" EX 300

# Good for: Data that changes gradually
# Risk: Stale data within TTL window
```

### Event-Based

```redis
# When product is updated:
DEL cache:api:products
DEL cache:product:123

# More aggressive: Delete all related caches
DEL cache:api:products:*    # Requires SCAN in practice
```

### Version-Based

```redis
# Include version in key
SET cache:products:v15 "[...]" EX 3600

# On data change, increment version
INCR cache:products:version  # Now v16
# Old cache automatically orphaned, will expire
```

## Cache Key Patterns

```redis
# API response caching
cache:api:/users:page:1:per_page:20
cache:api:/products?category=electronics&sort=price

# Query result caching
cache:query:SELECT_users_WHERE_active:hash123

# Computed values
cache:stats:daily:2024-01-15
cache:leaderboard:weekly
```

## Preventing Cache Stampede

When cache expires, many requests hit the database simultaneously:

```
┌──────────────────────────────────────────────────────┐
│  Cache expires at T=0                                │
│  T=0.001: Request 1 → cache miss → query DB         │
│  T=0.002: Request 2 → cache miss → query DB         │
│  T=0.003: Request 3 → cache miss → query DB         │
│  ... 100 simultaneous DB queries!                   │
└──────────────────────────────────────────────────────┘
```

### Solution: Locking

```redis
# First request acquires lock
SET cache:user:1001:lock "1" NX EX 10
# If acquired: query DB and update cache
# If not acquired: wait and retry GET
```

### Solution: Early Expiration

```redis
# Store with "soft" TTL in data
SET cache:data '{"value":"...","expires":1704067200}' EX 400

# Check soft expiry before hard expiry
# If soft expired, refresh asynchronously
```

## Caching Best Practices

1. **Set appropriate TTLs** - Balance freshness vs. hit rate
2. **Use meaningful keys** - Include parameters that affect the result
3. **Don't cache everything** - Focus on expensive, frequently-accessed data
4. **Monitor hit rates** - Track cache effectiveness
5. **Plan for cache failures** - App should work without cache (just slower)

📖 [Caching Patterns](https://redis.io/docs/latest/develop/use/patterns/)

## Resources

- [Redis Patterns](https://redis.io/docs/latest/develop/use/patterns/) — Common Redis usage patterns

---

> 📘 *This lesson is part of the [In-Memory Data Store Fundamentals](https://stanza.dev/courses/redis-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*