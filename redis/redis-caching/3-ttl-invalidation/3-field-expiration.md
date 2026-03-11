---
source_course: "redis-caching"
source_lesson: "redis-caching-hash-field-expiration"
---

# Hash Field Expiration

## Introduction
Redis 7.4 introduced per-field expiration for hashes — one of the most impactful caching features in years. Previously, you could only expire an entire hash key. Now you can set independent TTLs on individual hash fields, enabling fine-grained cache management within a single key. Redis 8 builds on this with convenience commands (HGETEX, HSETEX, HGETDEL) that combine read/write and expiration in a single call.

## Key Concepts
- **HEXPIRE**: Set a TTL in seconds on specific hash fields.
- **HPEXPIRE**: Same as HEXPIRE but with millisecond precision.
- **HTTL**: Check the remaining TTL of specific hash fields.
- **HPERSIST**: Remove expiration from hash fields, making them permanent.
- **HGETEX** (Redis 8): Fetch hash fields and optionally set their expiration in one command.
- **HSETEX** (Redis 8): Set hash fields and optionally set their expiration in one command.
- **HGETDEL** (Redis 8): Fetch and delete hash fields atomically.

## Real World Context
Consider a cached user profile stored as a hash with fields like `name`, `email`, `avatar_url`, and `recommendations`. The name and email rarely change (long TTL), but the avatar URL might be a CDN link that rotates hourly, and recommendations update every few minutes. Before Redis 8, you had to expire the entire hash or manage separate keys. Now each field can have its own lifecycle.

## Deep Dive

### Setting Per-Field Expiration

The HEXPIRE command sets a TTL on one or more hash fields:

```redis
# Store user profile as a hash
HSET cache:user:1001 name "Alice" email "a@ex.com" avatar_url "https://cdn/abc.jpg" recommendations "[...]"

# Expire avatar_url in 1 hour (3600 seconds)
HEXPIRE cache:user:1001 3600 FIELDS 1 avatar_url

# Expire recommendations in 5 minutes
HEXPIRE cache:user:1001 300 FIELDS 1 recommendations

# Set the overall hash to expire in 24 hours
EXPIRE cache:user:1001 86400
```

After 5 minutes, the `recommendations` field disappears while `name`, `email`, and `avatar_url` remain. After 1 hour, `avatar_url` also expires. The remaining fields live until the key-level TTL.

### Checking Field TTLs

Use HTTL to inspect remaining time:

```redis
# Check TTL of specific fields
HTTL cache:user:1001 FIELDS 2 avatar_url recommendations
# Output:
# 1) (integer) 3542   # avatar_url: ~59 minutes left
# 2) (integer) 245    # recommendations: ~4 minutes left

# Millisecond precision
HPTTL cache:user:1001 FIELDS 1 avatar_url
```

A return value of -1 means the field has no expiration. A return value of -2 means the field does not exist.

### Related Commands

| Command | Description |
|---------|-------------|
| HEXPIRE | Set field TTL in seconds |
| HPEXPIRE | Set field TTL in milliseconds |
| HEXPIREAT | Set field expiration as Unix timestamp (seconds) |
| HPEXPIREAT | Set field expiration as Unix timestamp (ms) |
| HTTL | Get remaining field TTL in seconds |
| HPTTL | Get remaining field TTL in milliseconds |
| HPERSIST | Remove expiration from fields |
| HGETEX | Get fields and optionally set expiration (Redis 8) |
| HSETEX | Set fields and optionally set expiration (Redis 8) |
| HGETDEL | Get and delete fields in one command (Redis 8) |

### Redis 8 Convenience Commands

Redis 8 added three commands that combine common hash operations with expiration:

```redis
# HGETEX: Read fields and set/refresh their TTL in one command
HGETEX cache:user:1001 EX 300 FIELDS 2 name email
# Returns field values AND refreshes their TTL to 5 minutes

# HSETEX: Write fields and set their TTL in one command
HSETEX cache:product:42 EX 3600 FIELDS 2 name Widget price 29.99
# Sets fields AND applies a 1-hour TTL in a single round-trip

# HGETDEL: Read and delete fields atomically
HGETDEL cache:user:1001 FIELDS 1 one_time_token
# Returns the value and removes it — ideal for single-use tokens
```

These commands eliminate the need for separate GET + EXPIRE or GET + DEL round-trips, reducing latency and race conditions.

### Practical Pattern: Layered Cache Freshness

```redis
# Product cache with different freshness requirements
HSET cache:product:42 name "Widget" description "A great widget" price "29.99" stock "142" trending_score "87"

# Price updates hourly from pricing service
HEXPIRE cache:product:42 3600 FIELDS 1 price

# Stock updates every 5 minutes from inventory
HEXPIRE cache:product:42 300 FIELDS 1 stock

# Trending score updates every minute
HEXPIRE cache:product:42 60 FIELDS 1 trending_score

# Static fields (name, description) follow the key-level TTL
EXPIRE cache:product:42 86400
```

This eliminates the need for separate keys per freshness tier and reduces both memory overhead and application complexity.

## Common Pitfalls
1. **Confusing key-level and field-level expiration** — EXPIRE sets TTL on the entire key. HEXPIRE sets TTL on individual fields within a hash. If the key expires, all fields are gone regardless of their field-level TTL.
2. **Not checking return values** — HEXPIRE returns different codes per field: 1 (expiration set successfully), 0 (condition not met, e.g. NX/XX/GT/LT), -2 (field does not exist), 2 (field deleted because the specified expiration is in the past). Always verify the operation succeeded.

## Best Practices
1. **Group fields by freshness tier** — Store fields with similar TTL requirements in the same hash and set appropriate per-field expirations.
2. **Set a key-level TTL as a safety net** — Even with per-field expiration, set EXPIRE on the hash itself to prevent orphaned fields from accumulating.

## Summary
- HEXPIRE/HPEXPIRE (Redis 7.4+) allow per-field TTLs within a single hash
- Redis 8 adds HGETEX, HSETEX, and HGETDEL for combined read/write + expiration
- HTTL/HPTTL let you inspect remaining time on individual fields
- This eliminates the need for multiple keys to represent objects with mixed freshness requirements
- Always combine field-level expiration with a key-level TTL safety net

## Code Examples

**Redis 8 per-field hash expiration — set, check, and remove TTLs on individual hash fields**

```bash
# Store user profile
HSET cache:user:1001 name "Alice" avatar_url "https://cdn/pic.jpg"

# Expire only avatar_url in 1 hour
HEXPIRE cache:user:1001 3600 FIELDS 1 avatar_url

# Check remaining TTL
HTTL cache:user:1001 FIELDS 1 avatar_url
# Output: (integer) 3598

# Remove expiration from a field
HPERSIST cache:user:1001 FIELDS 1 avatar_url
```


## Resources

- [HEXPIRE Command](https://redis.io/docs/latest/commands/hexpire/) — Official documentation for the HEXPIRE command

---

> 📘 *This lesson is part of the [High-Performance Caching Strategies](https://stanza.dev/courses/redis-caching) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*