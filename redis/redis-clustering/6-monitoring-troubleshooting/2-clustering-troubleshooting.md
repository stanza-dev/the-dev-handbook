---
source_course: "redis-clustering"
source_lesson: "redis-clustering-troubleshooting"
---

# Common Issues and Resolution

## Introduction

Diagnosing and resolving Redis cluster problems quickly is a critical operations skill. This lesson covers the most frequent issues with systematic diagnostic approaches and proven solutions.

## Key Concepts

- **MEMORY DOCTOR**: Built-in Redis command that analyzes memory usage patterns and suggests fixes
- **--bigkeys scan**: Redis CLI option that samples the keyspace to find the largest keys by type
- **SLOWLOG**: The internal log of commands exceeding a configurable execution time threshold
- **CLUSTER MEET**: Command to manually add a node to the cluster gossip network

## Real World Context

Production Redis issues typically fall into a few categories: memory pressure, high latency, cluster state failures, and replication lag. Knowing the diagnostic steps for each category reduces incident resolution time from hours to minutes.

## Deep Dive

### Issue: High Memory Usage

```redis
# Find memory-hungry keys
redis-cli --bigkeys
# Biggest string: user:session:abc (1024 bytes)
# Biggest list: queue:jobs (50000 items)

# Memory analysis
MEMORY DOCTOR
MEMORY STATS
MEMORY USAGE mykey SAMPLES 5
```

**Solutions:**
- Enable eviction policy
- Add TTL to keys
- Use more efficient data structures
- Scale horizontally (add nodes)

### Issue: High Latency

```redis
SLOWLOG GET 10
CONFIG SET slowlog-log-slower-than 10000  # 10ms
CONFIG SET slowlog-max-len 128
```

**Common causes:**
- Blocking commands (KEYS, FLUSHALL)
- Large key operations
- Memory fragmentation
- Slow client connections

### Issue: Cluster State FAIL

```redis
CLUSTER INFO
# cluster_state: fail

CLUSTER NODES
# Check node states
```

**Recovery:**
```redis
# If node is recoverable, rejoin cluster
CLUSTER MEET healthy-node-ip 6379

# If node is gone, perform failover on its replica
CLUSTER FAILOVER TAKEOVER
```

### Issue: Replication Lag

| Cause | Solution |
|-------|----------|
| High write volume | Add more replicas |
| Network issues | Check connectivity |
| Large RDB transfer | Use diskless replication |
| Slow replica | Upgrade replica hardware |

```redis
CONFIG SET repl-diskless-sync yes
CONFIG SET repl-diskless-sync-delay 5
```

### Issue: Connection Refused

```redis
INFO clients
# connected_clients: 10000
# rejected_connections: 500

CONFIG SET maxclients 20000
CONFIG SET tcp-keepalive 300
```

### Performance Tuning Checklist

```
- Disable THP (Transparent Huge Pages)
- Set vm.overcommit_memory = 1
- Configure appropriate maxmemory
- Use pipelining for batch operations
- Enable client-side caching where applicable
- Monitor and address slow queries
- Use SCAN instead of KEYS
- Set appropriate connection timeouts
```

```bash
# System tuning
echo never > /sys/kernel/mm/transparent_hugepage/enabled
echo 1 > /proc/sys/vm/overcommit_memory
sysctl -w net.core.somaxconn=65535
```

## Common Pitfalls

1. **Using KEYS in production to debug** -- KEYS blocks the server while scanning all keys. Use SCAN with a cursor for safe iteration.
2. **Not disabling Transparent Huge Pages** -- THP causes latency spikes in Redis due to memory allocation patterns. Disable it on all Redis servers.

## Best Practices

1. **Create runbooks for common issues** -- Document step-by-step procedures for each issue type so any team member can handle incidents.
2. **Use SCAN instead of KEYS** -- SCAN iterates incrementally without blocking, making it safe for production use.

## Summary

- Use `--bigkeys` and `MEMORY DOCTOR` to diagnose memory issues
- SLOWLOG identifies commands causing latency without MONITOR overhead
- Cluster FAIL state requires checking node status and performing failovers
- Diskless replication reduces initial sync time for large datasets
- Disable THP and tune OS settings for optimal Redis performance

## Code Examples

**Common troubleshooting commands and system tuning for Redis**

```bash
# Find large keys
redis-cli --bigkeys

# Check slow queries
redis-cli SLOWLOG GET 10

# Memory analysis
redis-cli MEMORY DOCTOR

# System tuning for Redis
echo never > /sys/kernel/mm/transparent_hugepage/enabled
echo 1 > /proc/sys/vm/overcommit_memory
```


## Resources

- [Redis Troubleshooting](https://redis.io/docs/latest/operate/oss_and_stack/management/troubleshooting/) — Redis troubleshooting guide

---

> 📘 *This lesson is part of the [Clustering and High Availability Architecture](https://stanza.dev/courses/redis-clustering) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*