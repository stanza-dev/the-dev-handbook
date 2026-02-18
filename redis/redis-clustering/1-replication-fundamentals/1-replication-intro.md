---
source_course: "redis-clustering"
source_lesson: "redis-clustering-replication-intro"
---

# Understanding Redis Replication

Replication in Redis allows you to create exact copies of your data on multiple servers. This provides high availability and read scalability.

## Why Replication?

```
┌─────────────────────────────────────────────────────────────┐
│  Without Replication:                                       │
│  Single Redis Server → Server fails → ALL DATA UNAVAILABLE │
├─────────────────────────────────────────────────────────────┤
│  With Replication:                                          │
│  Master ──▶ Replica 1                                       │
│         ──▶ Replica 2                                       │
│  Master fails → Promote Replica → DATA STILL AVAILABLE     │
└─────────────────────────────────────────────────────────────┘
```

**Benefits:**
- **High Availability**: Continue serving data if master fails
- **Read Scalability**: Distribute read load across replicas
- **Geographic Distribution**: Place replicas close to users
- **Backup**: Replicas serve as live backups

## Master-Replica Model

```
┌───────────────┐         ┌───────────────┐
│    MASTER     │         │   REPLICA     │
│               │────────▶│               │
│  Read/Write   │ (sync)  │  Read Only    │
│               │         │               │
└───────────────┘         └───────────────┘
        │
        │         ┌───────────────┐
        └────────▶│   REPLICA     │
           (sync) │               │
                  │  Read Only    │
                  └───────────────┘
```

- **Master**: Accepts all write operations
- **Replicas**: Read-only copies, receive updates from master
- **Asynchronous**: By default, replication is asynchronous (fast but eventual)

## How Replication Works

### Initial Synchronization

1. Replica connects to master
2. Master creates RDB snapshot
3. Master sends snapshot to replica
4. Replica loads snapshot
5. Master sends buffered commands since snapshot started

### Ongoing Replication

```
┌─────────────────────────────────────────────────────────────┐
│  Client: SET key "value"                                    │
│       ↓                                                     │
│  Master: Executes SET, returns OK to client                │
│       ↓                                                     │
│  Master: Sends SET command to all replicas (async)         │
│       ↓                                                     │
│  Replicas: Execute SET locally                             │
└─────────────────────────────────────────────────────────────┘
```

## Replication Lag

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

## Read Your Writes Concern

```
┌─────────────────────────────────────────────────────────────┐
│  Problem:                                                   │
│  Client writes to master → Client reads from replica       │
│  If read is faster than replication → Stale data!          │
├─────────────────────────────────────────────────────────────┤
│  Solutions:                                                 │
│  1. Always read from master (loses read scaling)           │
│  2. Use WAIT command for synchronous replication           │
│  3. Track versions/timestamps in application               │
└─────────────────────────────────────────────────────────────┘
```

## Synchronous Replication with WAIT

```redis
# Write and wait for 2 replicas to acknowledge
SET key "value"
WAIT 2 5000
# Wait for 2 replicas, timeout 5000ms
# Returns: number of replicas that acknowledged
```

**Note**: WAIT only waits for data to be received, not necessarily persisted.

📖 [Redis Replication](https://redis.io/docs/latest/operate/oss_and_stack/management/replication/)

## Resources

- [Redis Replication](https://redis.io/docs/latest/operate/oss_and_stack/management/replication/) — Official guide to Redis replication

---

> 📘 *This lesson is part of the [Clustering and High Availability Architecture](https://stanza.dev/courses/redis-clustering) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*