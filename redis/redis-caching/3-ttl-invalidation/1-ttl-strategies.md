---
source_course: "redis-caching"
source_lesson: "redis-caching-ttl-strategies"
---

# TTL Strategies

Choosing the right TTL (Time-To-Live) is crucial. Too short wastes cache benefits; too long serves stale data.

## Factors Affecting TTL Choice

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

📖 [EXPIRE Command](https://redis.io/docs/latest/commands/expire/)

---

> 📘 *This lesson is part of the [High-Performance Caching Strategies](https://stanza.dev/courses/redis-caching) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*