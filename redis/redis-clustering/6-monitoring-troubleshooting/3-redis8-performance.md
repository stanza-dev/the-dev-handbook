---
source_course: "redis-clustering"
source_lesson: "redis-clustering-redis8-performance"
---

# Performance Tuning for Redis 8

## Introduction

Redis 8 brings significant performance improvements including dual-stream replication and built-in data structures. I/O threading (available since Redis 6.0) can more than double throughput on multi-core CPUs. This lesson covers how to configure and optimize Redis 8 for maximum performance. Note that Redis Stack modules are now built into Redis 8 OSS as a single unified distribution.

## Key Concepts

- **io-threads**: Configuration parameter (available since Redis 6.0) that enables multi-threaded I/O for network operations
- **io-threads-do-reads**: Enables threading for read operations in addition to writes
- **Client-Side Caching**: Redis-assisted caching where the server notifies clients when cached keys are invalidated
- **Active Defragmentation**: Background process that compacts memory to reduce fragmentation

## Real World Context

Redis has traditionally been single-threaded for command execution. While command execution remains single-threaded, the I/O layer (reading from and writing to sockets) can be parallelized using I/O threads, a feature available since Redis 6.0. On servers with many concurrent connections, this eliminates the network I/O bottleneck and can improve throughput by up to 112%.

## Deep Dive

### I/O Threading Configuration

```bash
# Enable I/O threading (default: 1 = disabled)
io-threads 4

# Enable threaded reads (default: no)
io-threads-do-reads yes
```

**Guidelines:**
- Set `io-threads` to the number of CPU cores minus 1-2 (leave cores for OS and command execution)
- On a 8-core machine, try `io-threads 4` to start
- Benchmarks show up to 112% throughput improvement on multi-core CPUs
- Do not set io-threads higher than the number of available cores

### Memory Optimization

**Active Defragmentation:**
```redis
# Enable active defragmentation
CONFIG SET activedefrag yes

# Fragmentation threshold to start defrag (percentage)
CONFIG SET active-defrag-threshold-lower 10

# Fragmentation threshold for maximum effort
CONFIG SET active-defrag-threshold-upper 100

# CPU effort for defrag (percentage of CPU time)
CONFIG SET active-defrag-cycle-min 1
CONFIG SET active-defrag-cycle-max 25
```

**Memory-efficient data structures:**
```redis
# Use listpack encoding for small hashes (Redis 8 default)
# Automatically used when hash has <= 128 fields and values <= 64 bytes
hash-max-listpack-entries 128
hash-max-listpack-value 64

# Similar for sorted sets and lists
zset-max-listpack-entries 128
list-max-listpack-size -2
```

### Client-Side Caching

Redis 8 supports server-assisted client-side caching:

```redis
# Enable client tracking
CLIENT TRACKING ON REDIRECT <client-id>

# Or use broadcasting mode
CLIENT TRACKING ON BCAST PREFIX user: session:
```

```python
# Python example with redis-py
import redis

r = redis.Redis()

# Enable client-side caching
# The client library handles invalidation automatically
cache = r.cache()
value = cache.get('frequently_accessed_key')
# Subsequent reads served from local cache
# Server sends invalidation when key changes
```

### Connection Pooling Optimization

```python
import redis

# Optimized connection pool for Redis 8
pool = redis.ConnectionPool(
    host='localhost',
    port=6379,
    max_connections=50,
    socket_timeout=5,
    socket_connect_timeout=2,
    retry_on_timeout=True,
    health_check_interval=30
)
r = redis.Redis(connection_pool=pool)
```

### Benchmarking

```bash
# Baseline benchmark
redis-benchmark -h 127.0.0.1 -p 6379 -c 50 -n 100000

# Benchmark with pipeline
redis-benchmark -h 127.0.0.1 -p 6379 -c 50 -n 100000 -P 16

# Benchmark specific commands
redis-benchmark -h 127.0.0.1 -p 6379 -t set,get -n 100000
```

## Common Pitfalls

1. **Setting io-threads too high** -- More threads than CPU cores causes context switching overhead. On a 4-core machine, `io-threads 3` is the maximum useful setting.
2. **Enabling active defrag without monitoring** -- Active defragmentation uses CPU cycles. Monitor `active_defrag_running` and adjust cycle percentages if it impacts latency.

## Best Practices

1. **Benchmark before and after tuning** -- Use `redis-benchmark` to establish a baseline and measure the impact of each configuration change.
2. **Enable client-side caching for read-heavy workloads** -- For frequently-read keys, client-side caching eliminates network round-trips entirely.

## Summary

- Redis 8's `io-threads` can improve throughput by up to 112% on multi-core CPUs
- Redis Stack modules are now built into Redis 8 OSS (single unified distribution)
- Active defragmentation reduces memory waste without restarts
- Client-side caching eliminates network round-trips for frequently-read keys
- Always benchmark before and after configuration changes

## Code Examples

**Configure Redis 8 I/O threading and benchmark performance**

```bash
# Redis 8 I/O threading configuration
io-threads 4
io-threads-do-reads yes

# Benchmark to measure improvement
redis-benchmark -h 127.0.0.1 -p 6379 -c 50 -n 100000 -P 16

# Enable active defragmentation
redis-cli CONFIG SET activedefrag yes
```


## Resources

- [Redis Configuration](https://redis.io/docs/latest/operate/oss_and_stack/management/config/) — Redis server configuration reference

---

> 📘 *This lesson is part of the [Clustering and High Availability Architecture](https://stanza.dev/courses/redis-clustering) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*