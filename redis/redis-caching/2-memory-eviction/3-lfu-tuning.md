---
source_course: "redis-caching"
source_lesson: "redis-caching-lfu-tuning"
---

# LFU Tuning and Memory Analysis

## Introduction
The LFU eviction policy uses a frequency counter to decide which keys to evict. However, its default behavior may not suit every workload. Redis exposes tuning parameters and memory analysis tools that let you optimize eviction for your specific access patterns.

## Key Concepts
- **lfu-log-factor**: Controls how fast the frequency counter grows. Higher values slow counter growth, requiring more accesses before a key is considered 'hot'.
- **lfu-decay-time**: How often (in minutes) the frequency counter decays. Ensures keys that were popular in the past but are no longer accessed eventually get evicted.
- **OBJECT FREQ**: Returns the current LFU frequency counter for a key.

## Real World Context
Imagine an e-commerce site where certain products go viral temporarily. With default LFU settings, those products accumulate high frequency counters and remain cached long after the trend ends, wasting memory. Tuning the decay time ensures the cache adapts to changing popularity.

## Deep Dive

### LFU Configuration Parameters

These settings control LFU behavior in `redis.conf` or via CONFIG SET:

```bash
# How fast the counter grows (default: 10)
# Higher = more accesses needed to reach max frequency
lfu-log-factor 10

# Minutes between frequency counter decay (default: 1)
# 0 = never decay (not recommended for caches)
lfu-decay-time 1
```

The frequency counter is logarithmic — with `lfu-log-factor 10`, a key needs roughly 1 million accesses to reach the maximum counter value of 255. Lower values make the counter climb faster.

### Memory Analysis Tools

Redis provides several commands for understanding memory usage:

```redis
# Memory used by a specific key (in bytes)
MEMORY USAGE cache:user:1001
# Output: (integer) 156

# Diagnostic report
MEMORY DOCTOR

# Detailed memory statistics
MEMORY STATS

# LFU frequency counter of a key
OBJECT FREQ cache:user:1001
# Output: (integer) 42

# Idle time of a key (seconds since last access)
OBJECT IDLETIME cache:user:1001
```

The MEMORY USAGE command is particularly useful for estimating how much RAM your cached objects consume, helping you set an appropriate maxmemory limit.

### Tuning Strategy

A practical approach to tuning LFU:

1. Start with defaults (`lfu-log-factor 10`, `lfu-decay-time 1`)
2. Monitor with `OBJECT FREQ` on hot and cold keys
3. If cold keys have high counters, reduce `lfu-decay-time`
4. If hot keys get evicted too soon, increase `lfu-log-factor`

## Common Pitfalls
1. **Setting lfu-decay-time to 0** — The counter never decays, so keys that were popular once stay cached forever regardless of current access patterns. Always use a positive decay time.
2. **Not monitoring OBJECT FREQ** — Without checking frequency counters, you are tuning blind. Use OBJECT FREQ to verify your settings match actual behavior.

## Best Practices
1. **Use MEMORY USAGE before caching large objects** — Estimate the memory cost of caching a new data type before deploying to production.
2. **Combine LFU with TTL** — Even with LFU eviction, setting a TTL provides a safety net that guarantees data freshness.

## Summary
- lfu-log-factor controls counter growth speed; lfu-decay-time controls how fast popularity fades
- MEMORY USAGE, MEMORY DOCTOR, and OBJECT FREQ are essential diagnostic tools
- Start with defaults and tune based on observed frequency counters
- Always combine LFU eviction with TTL for a robust caching strategy

## Code Examples

**Diagnosing LFU behavior and tuning parameters at runtime**

```bash
# Check LFU frequency of a key
OBJECT FREQ cache:user:1001
# Output: (integer) 42

# Check memory used by a key
MEMORY USAGE cache:user:1001
# Output: (integer) 156

# Tune LFU at runtime
CONFIG SET lfu-log-factor 10
CONFIG SET lfu-decay-time 1
```


## Resources

- [Key Eviction](https://redis.io/docs/latest/develop/reference/eviction/) — Official guide to eviction policies and LFU tuning

---

> 📘 *This lesson is part of the [High-Performance Caching Strategies](https://stanza.dev/courses/redis-caching) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*