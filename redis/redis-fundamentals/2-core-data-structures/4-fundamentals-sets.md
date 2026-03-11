---
source_course: "redis-fundamentals"
source_lesson: "redis-fundamentals-sets"
---

# Sets: Unique Collections

## Introduction

Redis Sets are unordered collections of unique strings. They excel at membership testing, deduplication, and mathematical set operations like intersection and union — all in O(1) per element.

## Key Concepts

- **Uniqueness**: Adding the same value twice has no effect — Sets automatically deduplicate.
- **O(1) membership test**: SISMEMBER checks if an element exists in constant time.
- **Set operations**: SINTER (intersection), SUNION (union), and SDIFF (difference) combine multiple sets.
- **Random selection**: SRANDMEMBER and SPOP return random elements for sampling or lottery patterns.

## Real World Context

Sets track unique visitors per page, implement tag systems, model social graph relationships (followers, friends), and check online user status. SINTER finds mutual friends or shared tags between entities.

## Deep Dive

Redis Sets are unordered collections of unique strings. They're perfect for tracking unique items, tags, and performing mathematical set operations.

## Why Use Sets?

- **Uniqueness guaranteed**: Adding the same value twice has no effect
- **O(1) membership testing**: Check if an item exists instantly
- **Set operations**: Union, intersection, difference

## Basic Set Commands

### SADD and SMEMBERS

```redis
# Add members to a set
SADD tags "python" "redis" "backend"
# Returns: 3 (number added)

SADD tags "python"    # Adding duplicate
# Returns: 0 (already exists)

# Get all members
SMEMBERS tags
# Returns: ["python", "redis", "backend"] (unordered)

# Count members
SCARD tags           # Returns: 3
```

### Membership Testing

```redis
# Check if member exists (O(1) operation!)
SISMEMBER tags "python"    # Returns: 1 (true)
SISMEMBER tags "java"      # Returns: 0 (false)

# Check multiple members at once
SMISMEMBER tags "python" "java" "redis"
# Returns: [1, 0, 1]
```

## Removing Members

```redis
# Remove specific members
SREM tags "backend"
# Returns: 1 (removed)

# Remove and return random member
SPOP tags
# Returns: random member, removes it

SPOP tags 2          # Pop 2 random members
```

## Random Selection

```redis
# Get random member(s) without removing
SRANDMEMBER tags     # Returns: 1 random member
SRANDMEMBER tags 3   # Returns: 3 random members
```

## Set Operations

This is where Sets really shine!

### Intersection (members in ALL sets)

```redis
SADD set1 "a" "b" "c" "d"
SADD set2 "c" "d" "e" "f"

SINTER set1 set2
# Returns: ["c", "d"]

# Store result in new set
SINTERSTORE result set1 set2
# Creates "result" with ["c", "d"]
```

### Union (members in ANY set)

```redis
SUNION set1 set2
# Returns: ["a", "b", "c", "d", "e", "f"]

# Store result
SUNIONSTORE all_items set1 set2
```

### Difference (members in first but not others)

```redis
SDIFF set1 set2
# Returns: ["a", "b"] (in set1 but not set2)

SDIFF set2 set1
# Returns: ["e", "f"] (in set2 but not set1)
```

## Practical Examples

### Tracking Unique Visitors

```redis
# Track visitors per day
SADD visitors:2024-01-15 "user:1001"
SADD visitors:2024-01-15 "user:1002"
SADD visitors:2024-01-15 "user:1001"  # Duplicate ignored

# Count unique visitors
SCARD visitors:2024-01-15
# Returns: 2
```

### Tag System

```redis
# Articles with tags
SADD article:1:tags "redis" "database" "nosql"
SADD article:2:tags "redis" "caching" "performance"
SADD article:3:tags "database" "sql" "postgres"

# Find articles tagged with both "redis" and "database"
# (You'd need to maintain reverse indexes for this)
```

### Social Features

```redis
# User's followers
SADD user:1:followers "user:2" "user:3" "user:4"
SADD user:2:followers "user:1" "user:3" "user:5"

# Mutual followers (both follow each other)
SINTER user:1:followers user:2:followers
# Returns: ["user:3"]

# All unique people who follow either user
SUNION user:1:followers user:2:followers
```

### Online Status

```redis
# Mark user online
SADD online_users "user:1001"

# Check if online
SISMEMBER online_users "user:1001"

# Mark offline
SREM online_users "user:1001"

# Count online users
SCARD online_users
```

📖 [Set Commands Documentation](https://redis.io/docs/latest/commands/?group=set)

## Common Pitfalls

1. **Using Sets for ordered data** — Sets have no guaranteed order. Use sorted sets or lists if ordering matters.
2. **Expecting SMEMBERS to be fast on huge sets** — SMEMBERS returns ALL members (O(n)). For large sets, use SSCAN to iterate incrementally.

## Best Practices

1. **Use SISMEMBER for fast lookups** — O(1) membership checks are ideal for blacklists, whitelists, and feature flags.
2. **Leverage set operations** — SINTER and SUNION are powerful primitives. Use them to find mutual friends, common tags, or aggregate daily visitor sets into weekly totals.

## Summary

- Sets store unique, unordered string members.
- SISMEMBER provides O(1) membership testing.
- SADD returns 0 for duplicates, making it idempotent.
- SINTER, SUNION, and SDIFF perform mathematical set operations.
- SRANDMEMBER selects random elements without removing them.

## Code Examples

**Sets enforce uniqueness and support powerful operations — SINTER finds common members across sets**

```bash
# Track unique visitors
SADD visitors:today "user:1001" "user:1002" "user:1001"
SCARD visitors:today
# (integer) 2  (duplicate ignored)

# Find mutual followers
SADD user:1:follows "alice" "bob" "charlie"
SADD user:2:follows "bob" "charlie" "diana"
SINTER user:1:follows user:2:follows
# 1) "bob"
# 2) "charlie"
```


## Resources

- [Redis Sets](https://redis.io/docs/latest/develop/data-types/sets/) — Complete guide to Redis Set data type

---

> 📘 *This lesson is part of the [In-Memory Data Store Fundamentals](https://stanza.dev/courses/redis-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*