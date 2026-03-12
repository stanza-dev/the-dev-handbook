---
source_course: "redis-clustering"
source_lesson: "redis-clustering-network-partitioning"
---

# Network Partitioning and Consistency Trade-offs

## Introduction

Network partitions are inevitable in distributed systems. Understanding how Redis behaves during partitions and how to configure consistency guarantees helps you make informed trade-offs between availability and data safety.

## Key Concepts

- **CAP Theorem**: A distributed system can provide at most two of three guarantees: Consistency, Availability, Partition tolerance
- **Split-Brain**: When a network partition causes two independent groups to both accept writes
- **min-replicas-to-write**: Configuration that prevents the master from accepting writes when too few replicas are connected
- **cluster-require-full-coverage**: Whether the cluster refuses commands when some slots are uncovered

## Real World Context

Redis is an AP system by default (Availability + Partition tolerance), favoring availability over consistency. In many applications, losing a few seconds of writes during a partition is acceptable. But for financial or inventory systems, you may need to trade availability for consistency using `min-replicas-to-write`.

## Deep Dive

### CAP Theorem Applied to Redis

Redis defaults to AP behavior:
- **Availability**: Continues accepting writes on the majority side during partitions
- **Partition Tolerance**: Handles network splits via automatic failover
- **Consistency**: Sacrificed -- writes to the old master during a partition may be lost after failover

### The Consistency Window

When a master fails, there is a window where writes can be lost:

```
Timeline:
T0: Client writes to Master (acknowledged)
T1: Master fails before replicating to any replica
T2: Sentinel/Cluster promotes Replica (missing T0 write)
T3: Client connects to new Master -- T0 write is LOST
```

This window equals the time between the last successful replication and the failure.

### min-replicas Settings

```redis
# Master refuses writes unless N replicas are connected
min-replicas-to-write 1

# Maximum replication lag (seconds) for a replica to count
min-replicas-max-lag 10
```

**Behavior:**
- If fewer than `min-replicas-to-write` replicas have lag <= `min-replicas-max-lag`, the master returns errors for writes
- This limits the consistency window to `min-replicas-max-lag` seconds
- Trade-off: reduced availability (master stops accepting writes sooner)

### Split-Brain Scenarios

**Scenario 1: Sentinel with cross-region replicas**
```
Region A: Master + Sentinel 1
Region B: Replica + Sentinel 2 + Sentinel 3

Partition between regions:
- Region B promotes Replica (has quorum: 2/3 Sentinels)
- Region A's Master still accepts writes (no min-replicas check)
- Result: Two masters, diverging data
```

**Prevention:** Set `min-replicas-to-write 1` on the master. When the replica disconnects, the master stops accepting writes.

**Scenario 2: Redis Cluster partition**
```
Minority side: Masters without majority vote
- Returns CLUSTERDOWN for all commands
- No data divergence

Majority side: Promotes replicas, continues operating
```

Redis Cluster is safer than Sentinel in split-brain scenarios because it requires majority consensus.

### cluster-require-full-coverage

```redis
# Default: yes -- cluster stops if any slot is uncovered
cluster-require-full-coverage yes

# Alternative: no -- cluster serves requests for covered slots
cluster-require-full-coverage no
```

Setting `no` improves availability at the cost of partial data access during failures.

### WAIT for Per-Operation Consistency

```redis
SET critical_key "value"
WAIT 1 5000  # Wait for 1 replica, 5s timeout
# Returns 0 if no replica acknowledged (write may be lost)
# Returns 1 if at least 1 replica has the data
```

## Common Pitfalls

1. **Setting min-replicas-max-lag too low** -- A lag threshold of 1-2 seconds causes frequent write rejections during normal replication spikes. Use 10 seconds as a reasonable starting point.
2. **Expecting strong consistency from Redis** -- Redis is designed for performance and availability. If you need strict consistency, use WAIT for critical operations or consider a different database for that data.

## Best Practices

1. **Use min-replicas-to-write 1 for production** -- This is the simplest protection against split-brain data loss without significantly impacting availability.
2. **Combine WAIT with application-level retries** -- For critical writes, use WAIT and retry on the new master if the old one fails before acknowledgment.

## Summary

- Redis defaults to AP (availability over consistency) during partitions
- Writes acknowledged by the master but not replicated are lost during failover
- `min-replicas-to-write` trades availability for consistency by stopping writes when replicas disconnect
- Redis Cluster requires majority consensus, making it safer than Sentinel for split-brain
- Use WAIT for per-operation consistency on critical writes

## Code Examples

**Configure min-replicas for split-brain protection and use WAIT for critical writes**

```bash
# Configure split-brain protection
redis-cli CONFIG SET min-replicas-to-write 1
redis-cli CONFIG SET min-replicas-max-lag 10

# Per-operation consistency with WAIT
redis-cli SET critical_key "value"
redis-cli WAIT 1 5000
```


## Resources

- [Redis Cluster Consistency Guarantees](https://redis.io/docs/latest/operate/oss_and_stack/reference/cluster-spec/) — Redis Cluster specification including consistency guarantees

---

> 📘 *This lesson is part of the [Clustering and High Availability Architecture](https://stanza.dev/courses/redis-clustering) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*