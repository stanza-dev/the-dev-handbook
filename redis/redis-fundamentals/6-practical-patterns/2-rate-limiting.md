---
source_course: "redis-fundamentals"
source_lesson: "redis-fundamentals-rate-limiting"
---

# Rate Limiting Pattern

## Introduction

Rate limiting protects your APIs from abuse and overload. Redis is the perfect backend for rate limiters thanks to atomic counters, automatic expiration, and sub-millisecond operation — handling millions of rate-check decisions per second.

## Key Concepts

- **Fixed window**: Count requests in fixed time buckets (e.g., per minute) using INCR + EXPIRE NX.
- **Sliding window log**: Track exact request timestamps in a sorted set for precise rate limiting.
- **Token bucket**: Allow bursting while maintaining an average rate, best implemented with Lua scripts.
- **EXPIRE NX**: Sets TTL only if the key has none — essential for rate limit windows so the counter resets after the period.

## Real World Context

Every public API (GitHub, Stripe, Twitter) uses rate limiting. Without it, a single client can overwhelm your servers. Redis-backed rate limiters return rate limit headers (X-RateLimit-Remaining, X-RateLimit-Reset) to help clients throttle themselves.

## Deep Dive

Rate limiting protects your APIs from abuse. Redis makes it fast and simple to implement.

## Fixed Window Counter

The simplest approach - count requests in fixed time windows:

```redis
# On each request:
INCR rate:api:user:1001:minute
# First call creates key with value 1

# Set expiration only on first request
EXPIRE rate:api:user:1001:minute 60 NX
# NX = only if no TTL exists

# Check count
GET rate:api:user:1001:minute
# If > 100, reject request
```

### Implementation Logic

```
┌─────────────────────────────────────────────────────┐
│  Request comes in:                                   │
│  1. INCR rate:api:user:1001:minute                  │
│  2. If result > 100: REJECT (429 Too Many Requests) │
│  3. EXPIRE rate:api:user:1001:minute 60 NX          │
│  4. ALLOW request                                   │
└─────────────────────────────────────────────────────┘
```

## Sliding Window Log

More accurate - track exact timestamps of requests:

```redis
# Add request timestamp to sorted set
ZADD rate:user:1001 1704067200.123 "req:uuid1"
ZADD rate:user:1001 1704067201.456 "req:uuid2"

# Remove old entries (older than 60 seconds)
ZREMRANGEBYSCORE rate:user:1001 0 1704067140

# Count requests in window
ZCARD rate:user:1001
# If > 100, reject

# Set key expiration for cleanup
EXPIRE rate:user:1001 60
```

## Token Bucket Algorithm

Allows bursting while maintaining average rate:

```redis
# Initialize bucket (100 tokens, refills 10/sec)
HSET bucket:user:1001 tokens 100 last_update 1704067200

# On each request:
# 1. Calculate tokens to add since last_update
# 2. Add tokens (up to max)
# 3. If tokens > 0: decrement and allow
# 4. Else: reject

# Best implemented with Lua script for atomicity
```

## Practical Rate Limit Examples

### API Rate Limit by Endpoint

```redis
# Different limits per endpoint
INCR rate:api:/users:1001:minute     # General: 100/min
INCR rate:api:/search:1001:minute    # Search: 20/min
INCR rate:api:/upload:1001:hour      # Upload: 10/hour
```

### Tiered Rate Limits

```redis
# Free users: 100/hour
# Pro users: 1000/hour
# Enterprise: 10000/hour

# Check user tier, then appropriate key
GET rate:tier:free:user:1001:hour
GET rate:tier:pro:user:2001:hour
```

### IP-Based Rate Limiting

```redis
# Rate limit by IP (for unauthenticated requests)
INCR rate:ip:192.168.1.100:minute
EXPIRE rate:ip:192.168.1.100:minute 60 NX
```

## Rate Limit Response Headers

Good APIs return rate limit info in headers:

```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1704067260
```

```redis
# Get remaining
GET rate:api:user:1001:minute        # 5 used
# Remaining = 100 - 5 = 95

# Get reset time
TTL rate:api:user:1001:minute        # 45 seconds
# Reset = current_time + TTL
```

📖 [Rate Limiting Pattern](https://redis.io/docs/latest/develop/use/patterns/)

## Common Pitfalls

1. **Forgetting EXPIRE NX** — Using plain EXPIRE resets the window on every request, making the limit ineffective. NX ensures only the first request starts the timer.
2. **Race between INCR and EXPIRE** — If your app crashes between INCR and EXPIRE, the counter key lives forever. Use Lua scripts for atomicity in production.

## Best Practices

1. **Return rate limit headers** — Always include X-RateLimit-Limit, X-RateLimit-Remaining, and X-RateLimit-Reset so clients can self-throttle.
2. **Use tiered limits** — Different limits per user tier (free/pro/enterprise) and per endpoint (/search is more expensive than /users).

## Summary

- Fixed window (INCR + EXPIRE NX) is the simplest rate limiter.
- Sliding window log (sorted set) provides more accurate rate limiting.
- EXPIRE NX is critical — it prevents the TTL from resetting on each request.
- Return rate limit headers to help API consumers manage their usage.
- Use different limits per endpoint and user tier.

## Code Examples

**Fixed window rate limiting — INCR counts requests atomically, EXPIRE NX ensures the window resets after 60 seconds**

```bash
# Fixed window rate limit: 100 requests/minute
INCR rate:api:user:1001:minute
# Returns: current count

EXPIRE rate:api:user:1001:minute 60 NX
# NX = only set TTL if not already set

# Check: if count > 100, reject with 429
```


## Resources

- [Rate Limiting with Redis](https://redis.io/docs/latest/develop/use/patterns/) — Redis patterns for rate limiting and throttling

---

> 📘 *This lesson is part of the [In-Memory Data Store Fundamentals](https://stanza.dev/courses/redis-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*