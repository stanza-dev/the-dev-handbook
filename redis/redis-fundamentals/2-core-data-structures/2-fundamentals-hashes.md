---
source_course: "redis-fundamentals"
source_lesson: "redis-fundamentals-hashes"
---

# Hashes: Storing Objects

## Introduction

Redis Hashes are the go-to data type for representing objects with multiple fields. They are memory-efficient, allow access to individual fields without fetching the whole object, and now support per-field expiration in Redis 8.

## Key Concepts

- **Hash**: A mapping of field names to values, similar to a dictionary or object.
- **HSET/HGET**: Set and get individual fields within a hash.
- **HINCRBY**: Atomically increment a numeric field inside a hash.
- **Hash field expiration**: Redis 7.4+ lets you set TTL on individual fields with HEXPIRE, and Redis 8 adds HGETEX, HSETEX, and HGETDEL.

## Real World Context

Hashes are the natural choice for user profiles, product records, configuration objects — any entity with named properties. They use less memory than storing each field as a separate string key and are easier to manage as a logical unit.

## Deep Dive

Redis Hashes are perfect for representing objects with multiple fields. They're memory-efficient and allow you to access individual fields without fetching the entire object.

## Why Use Hashes?

Compare these two approaches:

```redis
# Approach 1: Multiple keys (inefficient)
SET user:1:name "Alice"
SET user:1:email "alice@example.com"
SET user:1:age "30"

# Approach 2: Hash (better!)
HSET user:1 name "Alice" email "alice@example.com" age "30"
```

Hashes are:
- More memory efficient
- Grouped logically
- Easier to manage (delete user:1 removes everything)

## Basic Hash Commands

### HSET and HGET

```redis
# Set single field
HSET user:1 name "Alice"
# Returns: 1 (new field created)

# Set multiple fields
HSET user:1 email "alice@example.com" age 30 country "USA"
# Returns: 3 (3 new fields)

# Get single field
HGET user:1 name
# Returns: "Alice"

# Get multiple fields
HMGET user:1 name email age
# Returns: ["Alice", "alice@example.com", "30"]

# Get all fields and values
HGETALL user:1
# Returns:
# 1) "name"
# 2) "Alice"
# 3) "email"
# 4) "alice@example.com"
# 5) "age"
# 6) "30"
# 7) "country"
# 8) "USA"
```

### Checking Fields

```redis
# Check if field exists
HEXISTS user:1 name    # Returns: 1
HEXISTS user:1 phone   # Returns: 0

# Get all field names
HKEYS user:1
# Returns: ["name", "email", "age", "country"]

# Get all values
HVALS user:1
# Returns: ["Alice", "alice@example.com", "30", "USA"]

# Count fields
HLEN user:1            # Returns: 4
```

## Numeric Operations on Fields

```redis
# Increment integer field
HSET product:1 views 100
HINCRBY product:1 views 1     # Returns: 101
HINCRBY product:1 views 10    # Returns: 111

# Increment float field
HSET product:1 price 19.99
HINCRBYFLOAT product:1 price 0.50   # Returns: "20.49"
```

## Hash Field Expiration (Redis 7.4+)

Redis 7.4 introduced per-field expiration, and Redis 8 added powerful new commands:

```redis
# Set field with expiration
HSET session:abc token "xyz123"
HEXPIRE session:abc 3600 FIELDS 1 token   # Expire in 1 hour

# Check field TTL
HTTL session:abc FIELDS 1 token
# Returns: [3598]
```

### New Redis 8 Hash Commands

Redis 8 introduces three commands that combine get/set with expiration or deletion:

```redis
# HGETEX: Get fields AND set their expiration atomically
HSET user:1 name "Alice" temp_code "xyz"
HGETEX user:1 EX 60 FIELDS 1 temp_code
# Returns: ["xyz"] — and temp_code now expires in 60 seconds

# HSETEX: Set fields AND set their expiration atomically
HSETEX user:1 EX 3600 FIELDS 2 session_token "abc123" csrf_token "def456"
# Both fields expire in 1 hour

# HGETDEL: Get fields AND delete them atomically
HGETDEL user:1 FIELDS 1 temp_code
# Returns: ["xyz"] — and temp_code is deleted
```

These commands simplify common caching and session patterns by eliminating the need for separate GET/EXPIRE or GET/DEL operations.

## Deleting Fields

```redis
# Delete specific fields
HDEL user:1 age country
# Returns: 2 (number of fields deleted)

# Delete entire hash
DEL user:1
```

## Practical Example: User Profile

```redis
# Create user profile
HSET user:1001 \
    username "alice_dev" \
    email "alice@example.com" \
    plan "premium" \
    created_at "2024-01-15" \
    login_count 0

# Increment login count on each login
HINCRBY user:1001 login_count 1

# Update plan
HSET user:1001 plan "enterprise"

# Get profile summary
HMGET user:1001 username email plan
```

📖 [Hash Commands Documentation](https://redis.io/docs/latest/commands/?group=hash)

## Common Pitfalls

1. **Using separate string keys instead of a hash** — `SET user:1:name`, `SET user:1:email` wastes memory and makes cleanup harder. Use `HSET user:1 name ... email ...` instead.
2. **Forgetting HGETALL is O(n)** — On hashes with many fields, HGETALL returns everything. Use HMGET to fetch only the fields you need.

## Best Practices

1. **Group related fields into one hash** — A single `user:1001` hash is more efficient and easier to manage than multiple string keys.
2. **Use Redis 8 HGETEX/HSETEX for session fields** — Set and read fields with automatic expiration in one atomic command.

## Summary

- Hashes store objects as field-value pairs within a single key.
- HSET sets multiple fields at once; HMGET reads specific fields.
- HINCRBY atomically increments numeric fields.
- Redis 8 adds HGETEX, HSETEX, and HGETDEL for atomic get/set with expiration or deletion.
- Hashes are more memory-efficient than multiple string keys for the same data.

## Code Examples

**Hash basics — HSET creates fields, HMGET reads multiple fields, HINCRBY atomically increments a numeric field**

```bash
# Store a user profile as a hash
HSET user:1001 name "Alice" email "alice@example.com" plan "premium"

# Read specific fields
HMGET user:1001 name plan
# 1) "Alice"
# 2) "premium"

# Increment a numeric field atomically
HINCRBY user:1001 login_count 1
```


## Resources

- [Redis Hashes](https://redis.io/docs/latest/develop/data-types/hashes/) — Complete guide to Redis Hash data type

---

> 📘 *This lesson is part of the [In-Memory Data Store Fundamentals](https://stanza.dev/courses/redis-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*