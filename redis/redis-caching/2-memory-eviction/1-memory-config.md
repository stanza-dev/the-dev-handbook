---
source_course: "redis-caching"
source_lesson: "redis-caching-memory-config"
---

# Memory Configuration

When using Redis as a cache, you must manage memory carefully. Redis stores everything in RAM, and running out of memory can crash your server.

## Setting Memory Limits

### In redis.conf

```bash
# Set maximum memory to 1GB
maxmemory 1gb

# Or in bytes
maxmemory 1073741824

# Other units: kb, mb, gb
maxmemory 500mb
```

### At Runtime

```redis
# Set limit dynamically
CONFIG SET maxmemory 1gb

# Check current setting
CONFIG GET maxmemory
```

## Monitoring Memory Usage

```redis
# Get memory info
INFO memory

# Key metrics:
# used_memory: Total bytes used by Redis
# used_memory_human: Human-readable format
# used_memory_peak: Maximum memory ever used
# maxmemory: Configured limit
# maxmemory_policy: Current eviction policy
```

Example output:
```
used_memory:1073741824
used_memory_human:1.00G
used_memory_peak:1288490188
maxmemory:2147483648
maxmemory_policy:allkeys-lru
```

## Memory Analysis

```redis
# Get memory usage of a specific key
MEMORY USAGE user:1001
# Returns: bytes used by the key

# Memory doctor (diagnostic report)
MEMORY DOCTOR

# Memory stats
MEMORY STATS
```

## What Happens When Memory is Full?

When Redis reaches maxmemory, behavior depends on the eviction policy:

1. **If eviction is enabled**: Redis removes keys to make room
2. **If eviction is disabled**: Redis returns errors for write commands

```
┌─────────────────────────────────────────────────────────────┐
│  Memory reaches maxmemory limit:                            │
│                                                             │
│  With eviction:                                             │
│  SET newkey "value" → Evicts old keys → Stores newkey       │
│                                                             │
│  Without eviction (noeviction):                             │
│  SET newkey "value" → (error) OOM command not allowed      │
│  GET existingkey → OK (reads still work)                   │
└─────────────────────────────────────────────────────────────┘
```

## Memory Optimization Tips

### Use Appropriate Data Types

```redis
# Bad: Multiple string keys
SET user:1001:name "Alice"
SET user:1001:email "alice@example.com"
SET user:1001:age "30"
# Memory: ~300 bytes (key overhead for each)

# Good: Single hash
HSET user:1001 name "Alice" email "alice@example.com" age 30
# Memory: ~150 bytes (shared key overhead)
```

### Keep Keys Short (But Readable)

```redis
# OK
user:1001:profile

# Too verbose
application:myapp:database:users:user_id:1001:profile_data

# Too cryptic
u:1001:p
```

### Use Compression for Large Values

```redis
# Store compressed JSON
SET cache:large_data "<gzip compressed data>"
```

### Set Appropriate TTLs

```redis
# Always expire cached data
SET cache:api:result "{...}" EX 300

# Prevents cache from growing indefinitely
```

📖 [Redis Memory Optimization](https://redis.io/docs/latest/operate/oss_and_stack/management/optimization/memory-optimization/)

## Resources

- [Memory Optimization](https://redis.io/docs/latest/operate/oss_and_stack/management/optimization/memory-optimization/) — Tips for reducing Redis memory usage

---

> 📘 *This lesson is part of the [High-Performance Caching Strategies](https://stanza.dev/courses/redis-caching) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*