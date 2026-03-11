---
source_course: "redis-fundamentals"
source_lesson: "redis-fundamentals-strings"
---

# Strings: The Foundation

## Introduction

Strings are the most basic and versatile Redis data type. Despite the name, they store any binary data up to 512 MB — text, numbers, serialized JSON, or raw bytes. Understanding strings is essential because they underpin counters, caches, locks, and many other patterns.

## Key Concepts

- **Binary-safe strings**: Redis strings can hold any data, not just text — including serialized objects and binary blobs.
- **Atomic operations**: INCR, DECR, and APPEND modify values in a single step with no race conditions.
- **Conditional writes**: NX (only if Not eXists) and XX (only if eXists) flags control when SET succeeds.
- **Bulk operations**: MSET and MGET handle multiple keys in a single round-trip.

## Real World Context

Strings power the most common Redis use cases: session tokens (SET with EX), page view counters (INCR), distributed locks (SET NX EX), and API response caches (SET with JSON payload and TTL).

## Deep Dive

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

## Common Pitfalls

1. **Storing large objects as strings** — A 512 MB string is possible but slow. For structured data, use hashes or JSON instead.
2. **Using GET then SET for updates** — This creates a race condition. Use INCR for counters or SET with GET flag for atomic read-and-replace.

## Best Practices

1. **Use INCR for counters, not GET+SET** — INCR is atomic and eliminates race conditions in concurrent environments.
2. **Leverage SET options** — Combine NX, EX, and GET in a single SET command instead of multiple calls.

## Summary

- Strings store any binary data up to 512 MB.
- INCR/DECR provide atomic numeric operations.
- SET NX EX creates distributed locks in one atomic command.
- MSET/MGET reduce round-trips for batch operations.
- Use GETSET or SET with GET for atomic read-and-replace.

## Code Examples

**Two essential String patterns — atomic counters with INCR and distributed locks with SET NX EX**

```bash
# Atomic counter pattern
SET page_views 0
INCR page_views       # 1
INCRBY page_views 10  # 11

# Distributed lock with SET NX EX
SET lock:order:5001 "worker:123" NX EX 30
# OK (acquired) or nil (already locked)
```


## Resources

- [Redis Strings](https://redis.io/docs/latest/develop/data-types/strings/) — Complete guide to Redis String data type

---

> 📘 *This lesson is part of the [In-Memory Data Store Fundamentals](https://stanza.dev/courses/redis-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*