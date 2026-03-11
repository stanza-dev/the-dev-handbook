---
source_course: "redis-fundamentals"
source_lesson: "redis-fundamentals-ttl-strategies"
---

# TTL and Expiration Strategies

## Introduction

Time-To-Live (TTL) makes keys self-cleaning — after a specified duration, Redis automatically deletes them. Mastering TTL strategies is essential for caching, session management, rate limiting, and any pattern involving temporary data.

## Key Concepts

- **EXPIRE/PEXPIRE**: Set timeout in seconds or milliseconds on an existing key.
- **TTL/PTTL**: Check remaining time; returns -1 (no TTL) or -2 (key gone).
- **PERSIST**: Remove expiration, making a key permanent again.
- **Conditional flags**: NX (only if no TTL), XX (only if TTL exists), GT (only if new TTL is greater), LT (only if new TTL is less).

## Real World Context

TTL drives every caching layer (5 minutes for API responses), every session store (30-minute sliding window), and every rate limiter (60-second counters). Without TTL, temporary data accumulates until Redis runs out of memory.

## Deep Dive

Time-To-Live (TTL) is a powerful feature that automatically removes keys after a specified time. Essential for caching, sessions, and temporary data.

## Setting Expiration

### When Creating Keys

```redis
# SET with expiration (seconds)
SET session:abc "user_data" EX 3600
# Expires in 1 hour

# SET with expiration (milliseconds)
SET cache:result "data" PX 30000
# Expires in 30 seconds

# SET with absolute Unix timestamp
SET event:ticket "valid" EXAT 1735689600
# Expires at specific time

# SETEX shorthand
SETEX session:xyz 3600 "user_data"
```

### On Existing Keys

```redis
# Set expiration in seconds
EXPIRE user:1001:cache 300
# Expires in 5 minutes

# Set expiration in milliseconds
PEXPIRE user:1001:cache 300000

# Set absolute expiration time
EXPIREAT user:1001:session 1735689600

# Set expiration only if key has no TTL
EXPIRE user:1001:data 3600 NX

# Set expiration only if key already has TTL
EXPIRE user:1001:data 7200 XX

# Set only if new TTL is greater than current
EXPIRE user:1001:data 7200 GT

# Set only if new TTL is less than current
EXPIRE user:1001:data 1800 LT
```

## Checking TTL

```redis
# Get remaining time in seconds
TTL user:1001:session
# Returns: 3542 (seconds remaining)
# Returns: -1 (no expiration set)
# Returns: -2 (key doesn't exist)

# Get remaining time in milliseconds
PTTL user:1001:session
# Returns: 3542000
```

## Removing Expiration

```redis
# Make key permanent
PERSIST user:1001:session
# Returns: 1 (removed TTL)
# Returns: 0 (no TTL existed)
```

## Expiration Strategies

### Cache with TTL

```redis
# Cache API response for 5 minutes
SET cache:api:users:page:1 "[{...}]" EX 300

# Check if cached, otherwise fetch and cache
GET cache:api:users:page:1
# If nil, fetch from DB and SET with EX
```

### Session Management

```redis
# Create session with 30-minute expiration
HSET session:abc123 user_id 1001 created_at 1704067200
EXPIRE session:abc123 1800

# Refresh session on activity (sliding expiration)
EXPIRE session:abc123 1800
```

### Rate Limiting with TTL

```redis
# Simple rate limit: 100 requests per minute
INCR rate:user:1001:minute
EXPIRE rate:user:1001:minute 60 NX   # Only set if no TTL

# Check count
GET rate:user:1001:minute
# If > 100, reject request
```

### Temporary Locks

```redis
# Acquire lock for 30 seconds
SET lock:resource:123 "owner:abc" NX EX 30
# NX = only if not exists
# EX 30 = auto-expire (prevents deadlocks)

# Release lock (only if we own it)
# Use Lua script in production for atomicity
```

## Hash Field Expiration (Redis 8+)

```redis
# Set TTL on individual hash fields
HSET user:1001 name "Alice" temp_token "xyz123"
HEXPIRE user:1001 3600 FIELDS 1 temp_token

# Check field TTL
HTTL user:1001 FIELDS 1 temp_token

# Persist a field (remove TTL)
HPERSIST user:1001 FIELDS 1 temp_token
```

📖 [EXPIRE Command](https://redis.io/docs/latest/commands/expire/)

## Common Pitfalls

1. **Separate SET and EXPIRE calls** — If your app crashes between SET and EXPIRE, the key lives forever. Use SET with EX in a single command.
2. **Forgetting EXPIRE NX for rate limiters** — Without NX, each request resets the TTL window. Use `EXPIRE key 60 NX` so only the first request sets the timer.

## Best Practices

1. **Use sliding TTL for sessions** — Call EXPIRE on every request to reset the timeout, creating an activity-based session window.
2. **Use SET with EX or PX** — Atomic set-with-expiration prevents orphaned keys from a crash between SET and EXPIRE.

## Summary

- EXPIRE sets automatic key deletion after N seconds.
- TTL returns remaining time (-1 = permanent, -2 = deleted).
- Use SET with EX for atomic set-and-expire.
- Use EXPIRE NX for rate limiters (only set TTL on first request).
- Redis 8 adds hash field expiration with HEXPIRE and HGETEX.

## Code Examples

**TTL lifecycle — set expiration with EX or EXPIRE, check with TTL, remove with PERSIST**

```bash
# Set expiration when creating a key
SET cache:data "value" EX 300

# Add expiration to an existing key
EXPIRE session:abc123 1800

# Check remaining TTL
TTL session:abc123
# (integer) 1798

# Remove expiration (make permanent)
PERSIST session:abc123
```


## Resources

- [Key Expiration](https://redis.io/docs/latest/commands/expire/) — EXPIRE command and TTL management

---

> 📘 *This lesson is part of the [In-Memory Data Store Fundamentals](https://stanza.dev/courses/redis-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*