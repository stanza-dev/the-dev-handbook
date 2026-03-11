---
source_course: "redis-fundamentals"
source_lesson: "redis-fundamentals-basic-commands"
---

# Essential Redis Commands

## Introduction

Redis commands are the building blocks of every interaction with the server. This lesson covers the essential commands for storing, retrieving, and managing keys — the foundation you will use throughout every Redis project.

## Key Concepts

- **SET/GET**: Store and retrieve string values by key.
- **INCR/DECR**: Atomically increment or decrement numeric values — no race conditions.
- **EXPIRE/TTL**: Set automatic key deletion after a timeout and check remaining time.
- **KEYS/EXISTS/DEL**: Discover, check, and remove keys from the database.

## Real World Context

These commands power the most common Redis patterns. SET/GET with EX handles session tokens, INCR drives page view counters and rate limiters, and EXPIRE ensures temporary data cleans itself up automatically.

## Deep Dive

Let's learn the fundamental commands that form the foundation of working with Redis.

## String Commands: SET and GET

Strings are the simplest Redis data type. Despite the name, they can store any data up to 512MB.

```redis
# Basic SET and GET
SET user:name "Alice"
GET user:name
# Returns: "Alice"

# SET with options
SET session:abc123 "user_data" EX 3600  # Expires in 3600 seconds
SET lock:resource "locked" NX           # Only set if NOT exists
SET counter "100" XX                     # Only set if EXISTS

# Get multiple keys at once
MGET user:name session:abc123
```

## Working with Numbers

Redis can treat strings as numbers for atomic operations:

```redis
# Increment and decrement
SET views 0
INCR views           # Returns: 1
INCR views           # Returns: 2
INCRBY views 10      # Returns: 12
DECR views           # Returns: 11

# Float operations
SET price 19.99
INCRBYFLOAT price 0.50   # Returns: "20.49"
```

## Key Management

```redis
# Check if key exists
EXISTS user:name     # Returns: 1 (exists) or 0 (not exists)

# Delete keys
DEL user:name        # Returns: 1 (deleted) or 0 (didn't exist)
DEL key1 key2 key3   # Delete multiple keys

# Get key type
TYPE user:name       # Returns: string

# Rename a key
RENAME oldkey newkey

# Find keys by pattern (use carefully in production!)
KEYS user:*          # All keys starting with "user:"
KEYS *session*       # All keys containing "session"
```

## Expiration (TTL)

Set automatic expiration on keys:

```redis
# Set expiration when creating
SET session:token "abc" EX 3600      # 3600 seconds (1 hour)
SET temp:data "xyz" PX 30000         # 30000 milliseconds (30 seconds)

# Set expiration on existing key
EXPIRE user:session 1800             # 30 minutes
PEXPIRE user:session 1800000         # 30 minutes in milliseconds

# Set absolute expiration time
EXPIREAT key 1735689600              # Unix timestamp

# Check remaining time
TTL user:session     # Returns seconds remaining, -1 if no expiry, -2 if key doesn't exist
PTTL user:session    # Returns milliseconds remaining

# Remove expiration (make permanent)
PERSIST user:session
```

## Useful String Operations

```redis
# Get string length
STRLEN user:name     # Returns: 5 (for "Alice")

# Append to string
APPEND user:name " Smith"
GET user:name        # Returns: "Alice Smith"

# Get substring
GETRANGE user:name 0 4   # Returns: "Alice"

# Set multiple keys at once
MSET user:1:name "Bob" user:1:email "bob@example.com"

# Get multiple keys
MGET user:1:name user:1:email
```

## Redis CLI Tips

```bash
# Get help for any command
127.0.0.1:6379> HELP SET
127.0.0.1:6379> HELP @string    # All string commands

# Clear screen
127.0.0.1:6379> CLEAR

# See all command groups
127.0.0.1:6379> HELP @generic
```

📖 [Redis Commands Reference](https://redis.io/docs/latest/commands/)

## Common Pitfalls

1. **Using KEYS in production** — `KEYS *` scans the entire keyspace and blocks Redis. Use SCAN for production key discovery.
2. **Forgetting that INCR creates keys** — INCR on a non-existent key creates it with value 0 then increments, which can lead to unexpected counters if your key naming has typos.

## Best Practices

1. **Prefer SET with EX/NX options** — Combine value setting with expiration in a single atomic command instead of separate SET + EXPIRE calls.
2. **Use MGET/MSET for batch operations** — Reduce network round-trips by getting or setting multiple keys in one command.

## Summary

- SET and GET are the most fundamental Redis commands.
- INCR provides atomic counters safe for concurrent access.
- EXPIRE adds automatic TTL-based cleanup to any key.
- TTL returns remaining seconds (-1 = no expiry, -2 = key missing).
- Use SCAN instead of KEYS in production environments.

## Code Examples

**Core Redis commands — SET/GET for strings, EX for expiration, INCR for atomic counters**

```bash
SET user:name "Alice"
GET user:name
# "Alice"

SET session:token "abc" EX 3600
TTL session:token
# (integer) 3600

INCR page_views
# (integer) 1
```


## Resources

- [Redis Commands](https://redis.io/docs/latest/commands/) — Complete reference for all Redis commands

---

> 📘 *This lesson is part of the [In-Memory Data Store Fundamentals](https://stanza.dev/courses/redis-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*