---
source_course: "redis-caching"
source_lesson: "redis-caching-monitoring-benchmarking"
---

# Monitoring and Benchmarking Cache Performance

## Introduction
You cannot improve what you do not measure. Redis provides built-in tools for monitoring cache hit rates, latency, and throughput. Combined with redis-benchmark, these tools help you validate your caching strategy and identify bottlenecks.

## Key Concepts
- **keyspace_hits / keyspace_misses**: Server-wide counters tracking cache hits and misses across all clients.
- **instantaneous_ops_per_sec**: Current throughput measured in operations per second.
- **SLOWLOG**: Records commands that exceed a configurable execution time threshold.

## Real World Context
A production Redis cache serving 50,000 requests per second needs continuous monitoring. A sudden drop in hit rate could indicate an eviction storm, a misconfigured TTL, or an upstream change that altered access patterns. Without metrics, these issues go undetected until users complain.

## Deep Dive

### INFO Stats for Cache Health

The INFO command provides essential cache metrics:

```redis
INFO stats
# keyspace_hits:9823451
# keyspace_misses:176549
# instantaneous_ops_per_sec:48230
# total_commands_processed:10000000
# evicted_keys:0
# expired_keys:234567
```

Calculate your hit rate from these counters: 9823451 / (9823451 + 176549) = 98.2%. An `evicted_keys` counter above zero means Redis is running out of memory and actively removing keys.

### Memory Monitoring

```redis
INFO memory
# used_memory_human:1.23G
# used_memory_peak_human:1.45G
# maxmemory_human:2.00G
# mem_fragmentation_ratio:1.12
```

A fragmentation ratio above 1.5 indicates significant memory fragmentation — consider restarting Redis or using MEMORY PURGE (Redis 4+).

### SLOWLOG

Identify slow commands that hurt cache performance:

```redis
# Configure threshold (microseconds)
CONFIG SET slowlog-log-slower-than 10000

# View slow commands
SLOWLOG GET 5
# Returns the 5 most recent slow commands with execution time

# Reset the log
SLOWLOG RESET
```

Common slow operations in caching: large MGET/MSET with thousands of keys, KEYS pattern scans, or serializing large objects.

### Benchmarking with redis-benchmark

Test your Redis instance throughput:

```bash
# Basic benchmark (default: 100k requests, 50 clients)
redis-benchmark

# Test specific commands
redis-benchmark -t set,get -n 1000000 -d 256
# -t: commands to test
# -n: total requests
# -d: data size in bytes

# Test with pipelining
redis-benchmark -t set,get -n 1000000 -P 16
# -P 16: pipeline 16 commands at once
```

Comparing results with and without pipelining shows the dramatic impact of reducing round-trips.

### Latency Monitoring

```bash
# Continuous latency monitoring
redis-cli --latency
# min: 0, max: 1, avg: 0.28 (1523 samples)

# Latency history (15-second intervals)
redis-cli --latency-history
```

If average latency exceeds 1ms on a local connection, investigate slow commands or memory pressure.

## Common Pitfalls
1. **Only checking hit rate at deploy time** — Cache performance degrades over time as data patterns shift. Monitor continuously, not just once.
2. **Ignoring evicted_keys** — A non-zero evicted_keys counter means your cache is too small for your dataset. Increase maxmemory or reduce what you cache.

## Best Practices
1. **Set up alerts on hit rate drops** — A hit rate below 80% usually indicates a problem worth investigating immediately.
2. **Benchmark before and after changes** — Always run redis-benchmark before and after configuration changes to measure the actual impact.

## Summary
- INFO stats provides hit rate, throughput, and eviction metrics
- SLOWLOG identifies commands that degrade cache performance
- redis-benchmark validates throughput with and without pipelining
- Monitor continuously — cache performance changes over time

## Code Examples

**Quick cache health check — hit rate from INFO stats and throughput benchmark with pipelining**

```bash
# Check cache hit rate from INFO stats
redis-cli INFO stats | grep keyspace
# keyspace_hits:9823451
# keyspace_misses:176549
# Hit rate: 9823451 / (9823451+176549) = 98.2%

# Benchmark GET/SET with pipelining
redis-benchmark -t set,get -n 100000 -P 16
```


## Resources

- [Redis Benchmark](https://redis.io/docs/latest/operate/oss_and_stack/management/optimization/benchmarks/) — Guide to using redis-benchmark for performance testing

---

> 📘 *This lesson is part of the [High-Performance Caching Strategies](https://stanza.dev/courses/redis-caching) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*