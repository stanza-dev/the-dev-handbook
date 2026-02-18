---
source_course: "redis-fundamentals"
source_lesson: "redis-fundamentals-strings"
---

# Strings: The Foundation

Strings are the most basic Redis data type. Despite the name, Redis strings can hold any data: text, numbers, serialized JSON, or even binary data up to 512 MB.

## String Basics

Strings are the building block for many Redis patterns:

```redis
# Simple string storage
SET greeting "Hello, Redis!"
GET greeting
# Returns: "Hello, Redis!"

# Strings can store any data
SET user:1001:json '{"name":"Alice","age":30}'
SET binary_data "\x00\x01\x02\x03"
```

## Atomic Counters

Redis strings support atomic numeric operations - perfect for counters:

```redis
# Initialize counter
SET page_views 0

# Atomic increment
INCR page_views       # Returns: 1
INCR page_views       # Returns: 2
INCR page_views       # Returns: 3

# Increment by specific amount
INCRBY page_views 10  # Returns: 13

# Decrement
DECR page_views       # Returns: 12
DECRBY page_views 5   # Returns: 7

# Float operations
SET price 19.99
INCRBYFLOAT price 0.50   # Returns: "20.49"
INCRBYFLOAT price -5.00  # Returns: "15.49"
```

**Key insight**: INCR on a non-existent key creates it with value 0, then increments.

## String Manipulation

```redis
# Append to string
SET message "Hello"
APPEND message ", World!"
GET message
# Returns: "Hello, World!"

# Get string length
STRLEN message        # Returns: 13

# Get substring (0-indexed)
GETRANGE message 0 4  # Returns: "Hello"
GETRANGE message -6 -1 # Returns: "World!"

# Overwrite part of string
SETRANGE message 7 "Redis!"
GET message
# Returns: "Hello, Redis!"
```

## Conditional SET Operations

```redis
# SET only if key does NOT exist (NX)
SET lock:resource "owner:123" NX
# Returns: OK (first time)
SET lock:resource "owner:456" NX
# Returns: (nil) - key already exists

# SET only if key EXISTS (XX)
SET existing_key "new_value" XX
# Only updates if key already exists

# GET old value while setting new (GETSET pattern)
GETSET counter 0
# Returns old value, sets new value atomically

# Redis 6.2+ alternative:
SET counter 100
SET counter 0 GET
# Returns: "100" (old value)
```

## Multiple Key Operations

```redis
# Set multiple keys atomically
MSET user:1:name "Alice" user:1:email "alice@example.com" user:1:age "30"

# Get multiple keys
MGET user:1:name user:1:email user:1:age
# Returns: ["Alice", "alice@example.com", "30"]

# MSETNX - Set multiple only if NONE exist
MSETNX a "1" b "2" c "3"
# All or nothing - fails if any key exists
```

## Binary-Safe Strings

Redis strings are binary-safe:

```redis
# Store binary data
SET image:thumbnail "<binary data>"

# Store serialized data
SET session:abc123 "<serialized session object>"

# Common serialization formats:
# - JSON (human readable, larger)
# - MessagePack (binary, smaller)
# - Protocol Buffers (binary, smallest)
```

## Practical Patterns

### Rate Limiter Counter

```redis
# Increment and set expiry atomically
SET rate:user:1001 1 EX 60 NX
# Or
INCR rate:user:1001
EXPIRE rate:user:1001 60 NX  # Only set TTL if not already set
```

### Distributed Lock

```redis
# Acquire lock (atomic)
SET lock:order:5001 "worker:123" NX EX 30
# NX = only if not exists
# EX 30 = expires in 30 seconds (prevents deadlock)
```

### Cache with Expiration

```redis
# Cache API response for 5 minutes
SET cache:api:/users '{"users":[...]}' EX 300
```

📖 [String Commands](https://redis.io/docs/latest/commands/?group=string)

## Resources

- [Redis Strings](https://redis.io/docs/latest/develop/data-types/strings/) — Complete guide to Redis String data type

---

> 📘 *This lesson is part of the [In-Memory Data Store Fundamentals](https://stanza.dev/courses/redis-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*