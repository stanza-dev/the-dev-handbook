---
source_course: "redis-caching"
source_lesson: "redis-caching-ttl-strategies"
---

# TTL Strategies

## Introduction
Choosing the right TTL (Time-To-Live) is crucial. Too short wastes cache benefits; too long serves stale data. A well-tuned TTL strategy balances freshness with performance.

## Key Concepts
- **TTL (Time-To-Live)**: The duration a cached entry remains valid before Redis automatically deletes it.
- **Jitter**: Random variation added to TTLs to prevent simultaneous expiration of many keys.
- **Soft Expiry**: A secondary threshold before the actual TTL where background refresh is triggered.

## Real World Context
An e-commerce site caching product prices needs short TTLs (minutes) to reflect frequent changes, while its category list can use long TTLs (hours) since categories rarely change. Getting these wrong means either stale prices or unnecessary database load.

## Deep Dive

### Factors Affecting TTL Choice

1. **Data volatility**: How often does the source data change?
2. **Staleness tolerance**: How much delay is acceptable?
3. **Cost of cache miss**: How expensive is regenerating data?
4. **Memory constraints**: Shorter TTLs = smaller cache footprint

## TTL Strategies by Data Type

### Static Content (Long TTL)

```redis
# Rarely changing data: 1 hour to 24 hours
SET cache:countries "[...json...]" EX 86400      # 24 hours
SET cache:categories "[...]" EX 3600             # 1 hour
SET cache:config "[...]" EX 86400                # 24 hours
```

### Semi-Static Content (Medium TTL)

```redis
# Changes occasionally: 5-60 minutes
SET cache:product:123 "{...}" EX 900             # 15 minutes
SET cache:user:profile:1001 "{...}" EX 1800      # 30 minutes
SET cache:exchange_rates "{...}" EX 300          # 5 minutes
```

### Dynamic Content (Short TTL)

```redis
# Changes frequently: seconds to minutes
SET cache:stock:price:AAPL "150.25" EX 30        # 30 seconds
SET cache:weather:NYC "{...}" EX 60              # 1 minute
SET cache:trending "[...]" EX 120                # 2 minutes
```

### Session Data (Activity-Based TTL)

```redis
# Sliding expiration - resets on activity
HSET session:abc123 user_id 1001
EXPIRE session:abc123 1800                       # 30 minutes

# On each request:
EXPIRE session:abc123 1800                       # Reset TTL
```

## Jittering TTLs to Prevent Stampedes

When many cache entries expire at the same time, you get a "thundering herd":

```
┌─────────────────────────────────────────────────────────────┐
│  Bad: All caches expire at T=300                            │
│                                                             │
│  T=300.001: Request 1 → MISS → Query DB                    │
│  T=300.002: Request 2 → MISS → Query DB                    │
│  T=300.003: Request 3 → MISS → Query DB                    │
│  ...100 simultaneous database queries!                     │
└─────────────────────────────────────────────────────────────┘
```

Solution: Add random jitter to TTLs:

```redis
# Instead of:
SET cache:product:1 "..." EX 300
SET cache:product:2 "..." EX 300
SET cache:product:3 "..." EX 300

# Use jittered TTLs (270-330 seconds):
SET cache:product:1 "..." EX 285   # 300 - random(0,30)
SET cache:product:2 "..." EX 312   # 300 + random(0,30)
SET cache:product:3 "..." EX 298   # 300 - random(0,30)
```

Implementation:

```python
import random

def jittered_ttl(base_ttl, jitter_percent=10):
    jitter = base_ttl * jitter_percent / 100
    return int(base_ttl + random.uniform(-jitter, jitter))

# Usage
ttl = jittered_ttl(300)  # Returns 270-330
```

## Early Expiration (Probabilistic)

Refresh cache before it expires to avoid misses:

```
┌─────────────────────────────────────────────────────────────┐
│  Cache TTL: 300 seconds                                     │
│                                                             │
│  Soft expiry: 270 seconds (90%)                            │
│  Hard expiry: 300 seconds                                   │
│                                                             │
│  If time > soft_expiry:                                     │
│    - Return cached data                                     │
│    - Asynchronously refresh cache                          │
└─────────────────────────────────────────────────────────────┘
```

Store expiration time in the value:

```redis
SET cache:data '{"value":"...","expires_at":1704067200}' EX 310

# On read, check expires_at
# If close to expiration, trigger background refresh
```

## Common Pitfalls
1. **Using the same TTL for all data** — Static configuration data and real-time stock prices need wildly different TTLs. Categorize data by volatility.
2. **Forgetting TTL jitter** — Without jitter, batch-cached data expires simultaneously, causing a thundering herd of database queries.

## Best Practices
1. **Add 10% jitter to all TTLs** — This simple technique prevents cache stampedes with minimal code.
2. **Use sliding TTLs for sessions** — Reset the TTL on each access so active sessions stay alive while idle ones expire.

## Summary
- Match TTL to data volatility: seconds for dynamic, hours for static
- Add jitter to prevent thundering herd on simultaneous expiration
- Use early/soft expiry to refresh cache before TTL hits zero
- Sliding TTLs work well for session data that resets on activity

📖 [EXPIRE Command](https://redis.io/docs/latest/commands/expire/)

## Code Examples

**Jittered TTL helper — adds ±10% randomness to prevent thousands of keys expiring simultaneously**

```python
import random

def jittered_ttl(base_ttl, jitter_percent=10):
    """Add random jitter to prevent thundering herd"""
    jitter = base_ttl * jitter_percent / 100
    return int(base_ttl + random.uniform(-jitter, jitter))

# Usage: TTLs spread between 270-330 seconds
ttl = jittered_ttl(300)  # e.g., 287, 312, 298...
r.set("cache:product:1", data, ex=ttl)
```


## Resources

- [EXPIRE Command](https://redis.io/docs/latest/commands/expire/) — Official documentation for key expiration

---

> 📘 *This lesson is part of the [High-Performance Caching Strategies](https://stanza.dev/courses/redis-caching) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*