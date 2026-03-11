---
source_course: "redis-caching"
source_lesson: "redis-caching-broadcast-mode"
---

# Broadcasting Mode and Cache Metrics

## Introduction
Redis client-side caching offers two tracking modes: default (per-key) and broadcasting. Each has different memory and network trade-offs. Understanding both modes and how to measure cache effectiveness is essential for production deployments.

## Key Concepts
- **Default Tracking Mode**: The server remembers which keys each client accessed and sends targeted invalidation messages. Uses server memory.
- **Broadcasting Mode (BCAST)**: The server does not track per-client keys. Instead, clients subscribe to key prefixes and receive all invalidation messages for matching keys. Uses no server memory.
- **Invalidation Table**: Server-side data structure that maps keys to clients in default mode.

## Real World Context
A microservice reading thousands of unique keys benefits from default mode — it only receives invalidations for keys it actually cached. A service that caches everything under a known prefix like `config:` benefits from broadcasting mode — simpler setup, no server-side tracking overhead.

## Deep Dive

### Default Mode vs Broadcasting Mode

```
┌─────────────────────────────────────────────────────────────┐
│  Default Mode:                                               │
│  Client A reads key1, key2                                  │
│  Server tracks: {key1 → [A], key2 → [A]}                   │
│  key1 modified → Server notifies only Client A              │
│  key3 modified → No notification (A never read key3)        │
│  ✓ Precise   ✗ Uses server memory                          │
├─────────────────────────────────────────────────────────────┤
│  Broadcasting Mode:                                          │
│  Client A subscribes to prefix "user:"                      │
│  Any key matching "user:*" modified → Server notifies A     │
│  ✓ No server memory   ✗ More network traffic               │
└─────────────────────────────────────────────────────────────┘
```

### Enabling Broadcasting Mode

```redis
# Default tracking (server remembers keys)
CLIENT TRACKING ON

# Broadcasting mode with prefix filters
CLIENT TRACKING ON BCAST PREFIX user: PREFIX product:

# Check tracking status
CLIENT TRACKINGINFO
```

The BCAST PREFIX option lets you subscribe to specific key prefixes, reducing noise from unrelated key changes.

### Monitoring Cache Metrics

Use INFO stats to track cache performance:

```redis
INFO stats
# tracking_clients: 12          (clients with tracking enabled)
# tracking_total_keys: 45231    (keys being tracked)
# tracking_total_items: 89102   (total client-key associations)
# tracking_total_prefixes: 3    (BCAST prefixes registered)

# Check a specific client's tracking
CLIENT LIST
CLIENT TRACKINGINFO
```

If `tracking_total_keys` is very large, consider switching to broadcasting mode to reduce server memory usage.

### Mode Selection Guide

| Criterion | Default Mode | Broadcasting Mode |
|-----------|-------------|-------------------|
| Server memory | High (tracks keys) | None |
| Network traffic | Low (targeted) | Higher (all prefixes) |
| Best for | Many unique keys | Known key prefixes |
| Setup complexity | Simple | Requires prefix planning |

## Common Pitfalls
1. **Using default mode with huge key spaces** — If clients read millions of unique keys, the server invalidation table consumes significant memory. Switch to BCAST mode for these workloads.
2. **Overly broad BCAST prefixes** — Using `BCAST PREFIX ""` (empty prefix) matches ALL keys, flooding the client with invalidations. Always use specific prefixes.

## Best Practices
1. **Monitor tracking_total_keys** — If it grows unbounded, your invalidation table is consuming too much server memory. Consider BCAST or limiting which keys are cached.
2. **Use BCAST for config and settings** — Application configuration cached under a known prefix like `config:` is an ideal BCAST use case.

## Summary
- Default mode tracks per-client keys for precise invalidation but uses server memory
- Broadcasting mode uses key prefix subscriptions with zero server memory overhead
- Use INFO stats and CLIENT TRACKINGINFO to monitor tracking overhead
- Choose default mode for diverse key access, BCAST for known prefix patterns

## Code Examples

**Setting up broadcasting mode with prefix-based invalidation subscriptions**

```bash
# Enable broadcasting mode with prefix filtering
CLIENT TRACKING ON BCAST PREFIX user: PREFIX product:

# Check current tracking configuration
CLIENT TRACKINGINFO
# Output:
# flags: B  (B = broadcasting mode)
# prefixes: user: product:
```


## Resources

- [Client-Side Caching Reference](https://redis.io/docs/latest/develop/reference/client-side-caching/) — Detailed reference for tracking modes and invalidation

---

> 📘 *This lesson is part of the [High-Performance Caching Strategies](https://stanza.dev/courses/redis-caching) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*