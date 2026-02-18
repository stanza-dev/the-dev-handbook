---
source_course: "redis-clustering"
source_lesson: "redis-clustering-troubleshooting"
---

# Common Issues and Resolution

Diagnose and resolve the most frequent Redis cluster problems.

## Issue: High Memory Usage

```redis
# Find memory-hungry keys
redis-cli --bigkeys

# Sample output:
# Biggest string: user:session:abc (1024 bytes)
# Biggest list: queue:jobs (50000 items)
# Biggest hash: cache:products (10000 fields)
```

```redis
# Memory analysis (Redis 4.0+)
MEMORY DOCTOR
MEMORY STATS

# Per-key memory usage
MEMORY USAGE mykey SAMPLES 5
```

**Solutions:**
- Enable eviction policy
- Add TTL to keys
- Use more efficient data structures
- Scale horizontally (add nodes)

## Issue: High Latency

```redis
# Check slow commands
SLOWLOG GET 10

# Output:
# 1) 1) (integer) 1              # ID
#    2) (integer) 1704067200     # Timestamp
#    3) (integer) 50000          # Duration (microseconds)
#    4) 1) "KEYS"                # Command
#       2) "*"
```

**Common causes:**
- Blocking commands (KEYS, FLUSHALL)
- Large key operations
- Memory fragmentation
- Slow client connections

```redis
# Configure slow log
CONFIG SET slowlog-log-slower-than 10000  # 10ms
CONFIG SET slowlog-max-len 128
```

## Issue: Cluster State FAIL

```redis
CLUSTER INFO
# cluster_state: fail

CLUSTER NODES
# Check node states
```

**Diagnosis:**

```bash
# Find which slots are uncovered
redis-cli CLUSTER SLOTS | grep -A2 "fail"

# Check specific node
redis-cli -h failed-node -p 6379 PING
```

**Recovery:**

```redis
# If node is recoverable, rejoin cluster
CLUSTER MEET healthy-node-ip 6379

# If node is gone, perform failover
# On a replica of the failed master:
CLUSTER FAILOVER TAKEOVER
```

## Issue: Replication Lag

```redis
INFO replication
# slave0:ip=...,state=online,offset=12345,lag=30
```

**Causes and solutions:**

| Cause | Solution |
|-------|----------|
| High write volume | Add more replicas |
| Network issues | Check connectivity |
| Large RDB transfer | Use diskless replication |
| Slow replica | Upgrade replica hardware |

```redis
# Enable diskless replication
CONFIG SET repl-diskless-sync yes
CONFIG SET repl-diskless-sync-delay 5
```

## Issue: Connection Refused

```redis
INFO clients
# connected_clients: 10000
# blocked_clients: 50
# rejected_connections: 500
```

**Solutions:**

```redis
# Increase max clients
CONFIG SET maxclients 20000

# Implement connection pooling in application
# Use TCP keepalive
CONFIG SET tcp-keepalive 300
```

## Performance Tuning Checklist

```
✓ Disable THP (Transparent Huge Pages)
✓ Set vm.overcommit_memory = 1
✓ Configure appropriate maxmemory
✓ Use pipelining for batch operations
✓ Enable client-side caching where applicable
✓ Monitor and address slow queries
✓ Use SCAN instead of KEYS
✓ Set appropriate connection timeouts
```

```bash
# System tuning
echo never > /sys/kernel/mm/transparent_hugepage/enabled
echo 1 > /proc/sys/vm/overcommit_memory
sysctl -w net.core.somaxconn=65535
```

📖 [Redis Troubleshooting](https://redis.io/docs/latest/operate/oss_and_stack/management/troubleshooting/)

## Resources

- [Redis Troubleshooting](https://redis.io/docs/latest/operate/oss_and_stack/management/troubleshooting/) — Redis troubleshooting guide

---

> 📘 *This lesson is part of the [Clustering and High Availability Architecture](https://stanza.dev/courses/redis-clustering) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*