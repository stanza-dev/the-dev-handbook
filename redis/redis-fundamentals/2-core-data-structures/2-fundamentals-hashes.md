---
source_course: "redis-fundamentals"
source_lesson: "redis-fundamentals-hashes"
---

# Hashes: Storing Objects

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

## Hash Field Expiration (Redis 8+)

Redis 8 introduces per-field expiration:

```redis
# Set field with expiration
HSET session:abc token "xyz123"
HEXPIRE session:abc 3600 FIELDS 1 token   # Expire in 1 hour

# Check field TTL
HTTL session:abc FIELDS 1 token
# Returns: [3598]
```

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

## Resources

- [Redis Hashes](https://redis.io/docs/latest/develop/data-types/hashes/) — Complete guide to Redis Hash data type

---

> 📘 *This lesson is part of the [In-Memory Data Store Fundamentals](https://stanza.dev/courses/redis-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*