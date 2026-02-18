---
source_course: "redis-caching"
source_lesson: "redis-caching-eviction-policies"
---

# Eviction Policies

When Redis runs out of memory, it needs to decide which keys to remove. The eviction policy controls this decision.

## Available Policies

### noeviction

```bash
maxmemory-policy noeviction
```

- **Behavior**: Returns errors when memory is full
- **Use case**: When data loss is unacceptable
- **Reads**: Always work
- **Writes**: Fail with `OOM` error

### allkeys-lru

```bash
maxmemory-policy allkeys-lru
```

- **Behavior**: Evicts least recently used keys from ALL keys
- **Use case**: General caching (most common for caches)
- **Algorithm**: Approximated LRU using sampling

### volatile-lru

```bash
maxmemory-policy volatile-lru
```

- **Behavior**: Evicts LRU keys only from keys with TTL set
- **Use case**: Mix of cache (with TTL) and persistent data (no TTL)

### allkeys-lfu

```bash
maxmemory-policy allkeys-lfu
```

- **Behavior**: Evicts least frequently used keys
- **Use case**: When access frequency matters more than recency
- **Better for**: Data with varying popularity

### volatile-lfu

```bash
maxmemory-policy volatile-lfu
```

- **Behavior**: LFU eviction only for keys with TTL
- **Use case**: Same as volatile-lru but frequency-based

### allkeys-random

```bash
maxmemory-policy allkeys-random
```

- **Behavior**: Randomly evicts keys
- **Use case**: When all keys have equal importance
- **Fastest**: No tracking overhead

### volatile-random

```bash
maxmemory-policy volatile-random
```

- **Behavior**: Random eviction from keys with TTL

### volatile-ttl

```bash
maxmemory-policy volatile-ttl
```

- **Behavior**: Evicts keys with shortest remaining TTL
- **Use case**: When short TTL indicates less importance

## LRU vs LFU Comparison

```
┌─────────────────────────────────────────────────────────────┐
│  LRU (Least Recently Used):                                 │
│  Evicts keys not accessed for the longest time              │
│                                                             │
│  Timeline: ────────────────────────────────────────→        │
│  Key A:    [accessed]                      [evict?]         │
│  Key B:              [accessed]            [keep]           │
│                                                             │
│  Good for: General caching, recent = relevant               │
├─────────────────────────────────────────────────────────────┤
│  LFU (Least Frequently Used):                               │
│  Evicts keys accessed the fewest times                      │
│                                                             │
│  Key A:    [1 access]                      [evict?]         │
│  Key B:    [100 accesses]                  [keep]           │
│                                                             │
│  Good for: Popular content should stay cached               │
└─────────────────────────────────────────────────────────────┘
```

## Configuring LFU

```bash
# LFU tuning parameters in redis.conf
lfu-log-factor 10      # Higher = slower frequency counter growth
lfu-decay-time 1       # Minutes between frequency counter decay
```

## Checking Current Policy

```redis
# Get current policy
CONFIG GET maxmemory-policy

# Change policy at runtime
CONFIG SET maxmemory-policy allkeys-lru
```

## Eviction Policy Selection Guide

| Scenario | Recommended Policy |
|----------|-------------------|
| Pure cache (all data can be regenerated) | allkeys-lru or allkeys-lfu |
| Cache + persistent data mixed | volatile-lru |
| All keys equally important | allkeys-random |
| Can't afford data loss | noeviction |
| Short-lived data is less valuable | volatile-ttl |
| Popularity matters (hot content) | allkeys-lfu |

📖 [Key Eviction](https://redis.io/docs/latest/develop/reference/eviction/)

## Resources

- [Key Eviction](https://redis.io/docs/latest/develop/reference/eviction/) — Complete guide to Redis eviction policies

---

> 📘 *This lesson is part of the [High-Performance Caching Strategies](https://stanza.dev/courses/redis-caching) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*