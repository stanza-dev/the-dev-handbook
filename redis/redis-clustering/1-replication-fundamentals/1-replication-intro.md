---
source_course: "redis-clustering"
source_lesson: "redis-clustering-replication-intro"
---

# Understanding Redis Replication

## Introduction

Replication in Redis allows you to create exact copies of your data on multiple servers. This provides high availability and read scalability, forming the foundation of every production Redis deployment.

## Key Concepts

- **Master**: The primary server that accepts all write operations
- **Replica**: A read-only copy that receives updates from the master
- **Asynchronous Replication**: By default, replication is asynchronous (fast but eventual)
- **Replication Lag**: The delay between a write on the master and its appearance on the replica

## Real World Context

Without replication, a single Redis server failure means total data unavailability. In production, every Redis deployment should have at least one replica for failover capability and read scaling.

## Deep Dive

### Why Replication?

```
+---------------------------------------------------------------+
|  Without Replication:                                         |
|  Single Redis Server -> Server fails -> ALL DATA UNAVAILABLE  |
+---------------------------------------------------------------+
|  With Replication:                                            |
|  Master --> Replica 1                                         |
|         --> Replica 2                                         |
|  Master fails -> Promote Replica -> DATA STILL AVAILABLE     |
+---------------------------------------------------------------+
```

**Benefits:**
- **High Availability**: Continue serving data if master fails
- **Read Scalability**: Distribute read load across replicas
- **Geographic Distribution**: Place replicas close to users
- **Backup**: Replicas serve as live backups

### Master-Replica Model

- **Master**: Accepts all write operations
- **Replicas**: Read-only copies, receive updates from master
- **Asynchronous**: By default, replication is asynchronous (fast but eventual)

### How Replication Works

**Initial Synchronization:**

1. Replica connects to master
2. Master creates RDB snapshot
3. Master sends snapshot to replica
4. Replica loads snapshot
5. Master sends buffered commands since snapshot started

**Ongoing Replication:**

```
+---------------------------------------------------------------+
|  Client: SET key "value"                                      |
|       v                                                       |
|  Master: Executes SET, returns OK to client                   |
|       v                                                       |
|  Master: Sends SET command to all replicas (async)            |
|       v                                                       |
|  Replicas: Execute SET locally                                |
+---------------------------------------------------------------+
```

### Replication Lag

Since replication is asynchronous, replicas may be slightly behind:

```redis
# Check replication status
INFO replication

# On master, shows:
# role:master
# connected_slaves:2
# slave0:ip=192.168.1.11,port=6379,state=online,offset=12345,lag=0
# slave1:ip=192.168.1.12,port=6379,state=online,offset=12340,lag=1
```

**Lag** is the number of seconds since the replica last communicated.

### Read Your Writes Concern

If a client writes to master then immediately reads from a replica, it may see stale data. Solutions include:
1. Always read from master (loses read scaling)
2. Use WAIT command for synchronous replication
3. Track versions/timestamps in application

### Synchronous Replication with WAIT

```redis
# Write and wait for 2 replicas to acknowledge
SET key "value"
WAIT 2 5000
# Wait for 2 replicas, timeout 5000ms
# Returns: number of replicas that acknowledged
```

**Note**: WAIT only waits for data to be received, not necessarily persisted.

### Redis 8 Dual-Stream Replication

Redis 8 introduces a new dual-stream replication mechanism that significantly improves replication performance. Benchmarks show 35% lower peak replication buffer usage, 18% faster replication time, and 7.5% higher write rate during active replication compared to Redis 7. This is especially impactful for deployments with large datasets or high write throughput.

## Common Pitfalls

1. **Reading from replicas too early** -- Asynchronous replication means replicas may not yet have the latest writes. Use WAIT or read from master for critical reads.
2. **Ignoring replication lag** -- Monitor `master_last_io_seconds_ago` on replicas. Lag above 10 seconds usually indicates network or performance issues.

## Best Practices

1. **Always set up at least one replica** -- Even for development, having a replica helps catch replication-related bugs early.
2. **Use WAIT for critical writes** -- When data loss is unacceptable, use WAIT to ensure at least one replica has acknowledged the write.

## Summary

- Replication creates read-only copies of your data on multiple servers
- It is asynchronous by default, with optional synchronous behavior via WAIT
- Redis 8 introduces dual-stream replication for 18% faster sync times
- Monitor replication lag to detect issues early
- Replicas can serve reads but may return slightly stale data

## Code Examples

**Check replication status and use WAIT for synchronous replication**

```bash
# Check replication status on master
redis-cli INFO replication

# Synchronous write with WAIT
redis-cli SET important_key "critical_value"
redis-cli WAIT 2 5000
```


## Resources

- [Redis Replication](https://redis.io/docs/latest/operate/oss_and_stack/management/replication/) — Official guide to Redis replication

---

> 📘 *This lesson is part of the [Clustering and High Availability Architecture](https://stanza.dev/courses/redis-clustering) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*