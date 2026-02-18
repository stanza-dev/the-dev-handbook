---
source_course: "redis-fundamentals"
source_lesson: "redis-fundamentals-ttl-strategies"
---

# TTL and Expiration Strategies

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

## Resources

- [Key Expiration](https://redis.io/docs/latest/commands/expire/) — EXPIRE command and TTL management

---

> 📘 *This lesson is part of the [In-Memory Data Store Fundamentals](https://stanza.dev/courses/redis-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*