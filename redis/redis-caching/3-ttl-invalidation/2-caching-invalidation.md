---
source_course: "redis-caching"
source_lesson: "redis-caching-invalidation"
---

# Cache Invalidation Strategies

Cache invalidation is challenging but essential. Here are proven strategies.

## Strategy 1: Time-Based Invalidation (TTL)

```redis
# Let caches expire naturally
SET cache:data "..." EX 300

# Pros: Simple, automatic
# Cons: Data may be stale until expiration
```

## Strategy 2: Active Invalidation

Delete cache when source data changes:

```redis
# On database UPDATE/INSERT/DELETE:
DEL cache:user:1001
DEL cache:user:1001:profile
DEL cache:api:users:list
```

### Pattern: Namespace Invalidation

When updating data affects multiple cache keys:

```redis
# Product update affects multiple caches
# Option A: Delete all related keys
DEL cache:product:123
DEL cache:products:category:electronics
DEL cache:products:featured
DEL cache:search:electronics

# Option B: Use SCAN to find and delete
SCAN 0 MATCH cache:product:123:* COUNT 100
# Delete each returned key
```

### Pattern: Event-Driven Invalidation

Publish invalidation events:

```redis
# When data changes, publish event
PUBLISH cache:invalidation '{"type":"product","id":123}'

# Subscribers delete their caches
# SUB receives message → DEL cache:product:123
```

## Strategy 3: Version-Based Invalidation

Include version in cache key:

```redis
# Store version
SET product:123:version 5

# Cache with version in key
SET cache:product:123:v5 "{...}" EX 3600

# On update:
INCR product:123:version  # Now v6
# Old cache (v5) is orphaned, will expire
# New requests use v6 key
```

**Pros**: Clean invalidation, no explicit delete needed
**Cons**: Orphaned keys waste memory until TTL

## Strategy 4: Generation/Epoch-Based

For invalidating all caches at once:

```redis
# Store cache generation
SET cache:generation 42

# Include generation in all cache keys
SET cache:g42:user:1001 "..." EX 3600

# To invalidate ALL caches:
INCR cache:generation  # Now 43
# All old g42 caches are orphaned
```

## Handling Cache Stampede

When cache is invalidated, multiple requests may try to rebuild it:

### Solution 1: Locking

```redis
# First request acquires lock
SET cache:user:1001:lock "1" NX EX 10
# If acquired:
#   - Query database
#   - Update cache
#   - Delete lock
# If not acquired:
#   - Wait briefly
#   - Retry GET from cache
```

### Solution 2: Stale-While-Revalidate

```redis
# Store with soft and hard TTL
SET cache:data '{"value":"...","soft_ttl":1704067200}' EX 400

# On read:
# If past soft_ttl:
#   - Return stale data immediately
#   - Trigger async refresh
```

## Invalidation Patterns Summary

| Pattern | Complexity | Consistency | Use Case |
|---------|------------|-------------|----------|
| TTL Only | Low | Eventual | Tolerance for staleness |
| Active DEL | Medium | Strong | Write-heavy apps |
| Version-Based | Medium | Strong | Clean invalidation needed |
| Event-Driven | High | Strong | Distributed systems |

📖 [Caching Best Practices](https://redis.io/docs/latest/develop/use/patterns/)

---

> 📘 *This lesson is part of the [High-Performance Caching Strategies](https://stanza.dev/courses/redis-caching) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*