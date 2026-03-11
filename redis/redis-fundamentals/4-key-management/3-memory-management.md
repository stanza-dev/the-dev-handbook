---
source_course: "redis-fundamentals"
source_lesson: "redis-fundamentals-memory-management"
---

# Memory Management and Eviction

## Introduction

Redis stores everything in memory, so managing memory is critical. When memory runs out, Redis must decide what to do — reject new writes or evict existing keys. Understanding eviction policies lets you control this behavior.

## Key Concepts

- **maxmemory**: The maximum amount of memory Redis is allowed to use. Once reached, the eviction policy kicks in.
- **Eviction policy**: The algorithm Redis uses to decide which keys to remove when memory is full.
- **Volatile keys**: Keys that have an expiration (TTL) set.
- **LRU vs LFU**: Least Recently Used evicts keys not accessed recently; Least Frequently Used evicts keys accessed least often overall.

## Real World Context

Without a maxmemory limit, Redis grows until the OS kills it (OOM). In production, you always set maxmemory and choose an eviction policy that matches your use case — caches use allkeys-lru, while mixed workloads may use volatile-lru to protect persistent keys.

## Deep Dive

### Setting Memory Limits

```redis
# Set max memory to 256MB
CONFIG SET maxmemory 256mb

# Check current memory usage
INFO memory
# used_memory_human: 45.12M
# maxmemory_human: 256.00M
# maxmemory_policy: noeviction

# Set in redis.conf for persistence
maxmemory 256mb
```

### Eviction Policies

| Policy | Scope | Algorithm | Best For |
|--------|-------|-----------|----------|
| noeviction | — | Reject writes | Primary database |
| allkeys-lru | All keys | Least Recently Used | General cache |
| volatile-lru | Keys with TTL | Least Recently Used | Mixed workload |
| allkeys-lfu | All keys | Least Frequently Used | Frequency-based cache |
| volatile-lfu | Keys with TTL | Least Frequently Used | Popular item cache |
| allkeys-random | All keys | Random | When all keys are equal |
| volatile-random | Keys with TTL | Random | Simple TTL cache |
| volatile-ttl | Keys with TTL | Shortest TTL first | Time-sensitive data |

```redis
# Set eviction policy
CONFIG SET maxmemory-policy allkeys-lru

# Check current policy
CONFIG GET maxmemory-policy
```

### Monitoring Memory

```redis
# Detailed memory stats
INFO memory

# Key metrics:
# used_memory: Total bytes allocated
# used_memory_peak: Maximum bytes ever used
# mem_fragmentation_ratio: > 1.5 means fragmentation
# evicted_keys: Number of keys evicted

# Memory usage for a specific key
MEMORY USAGE user:1001
# Returns: 72 (bytes)

# Memory doctor (diagnostics)
MEMORY DOCTOR
```

### Memory Optimization Tips

```redis
# Use hashes for small objects (ziplist encoding)
# Instead of:
SET user:1:name "Alice"
SET user:1:email "alice@example.com"

# Use a hash (more memory efficient):
HSET user:1 name "Alice" email "alice@example.com"

# Short keys save memory at scale
SET u:1:n "Alice"    # Saves bytes per key
```

## Common Pitfalls

1. **Not setting maxmemory** — Redis grows until the OS kills it with an OOM error. Always set a limit in production.
2. **Using noeviction for caches** — With noeviction, Redis returns errors when full instead of evicting old data. Use allkeys-lru or allkeys-lfu for caches.
3. **Ignoring fragmentation** — A mem_fragmentation_ratio above 1.5 wastes memory. Restart Redis or enable active defragmentation.

## Best Practices

1. **Set maxmemory to 75% of available RAM** — Leave room for the OS, background saves (fork), and other processes.
2. **Match policy to workload** — Use allkeys-lru for pure caches, volatile-lru when mixing cached and persistent data, and noeviction when every key matters.
3. **Monitor evicted_keys** — A rising count means Redis is under memory pressure. Scale up or optimize data structures.

## Summary

- Always set maxmemory in production to prevent OOM kills.
- Choose an eviction policy that matches your use case (allkeys-lru for caches, noeviction for databases).
- Monitor memory with INFO memory and watch for fragmentation.
- Use memory-efficient data structures like hashes for small objects.

## Code Examples

**Checking Redis memory stats — monitor used_memory, maxmemory, and evicted_keys to ensure healthy memory usage**

```bash
# Check memory usage and eviction policy
redis-cli INFO memory | grep -E 'used_memory_human|maxmemory|evicted_keys'
# used_memory_human:45.12M
# maxmemory_human:256.00M
# maxmemory_policy:allkeys-lru
# evicted_keys:0
```


## Resources

- [Redis Memory Optimization](https://redis.io/docs/latest/operate/oss_and_stack/management/optimization/memory-optimization/) — Official guide to optimizing Redis memory usage

---

> 📘 *This lesson is part of the [In-Memory Data Store Fundamentals](https://stanza.dev/courses/redis-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*