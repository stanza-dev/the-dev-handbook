---
source_course: "redis-clustering"
source_lesson: "redis-clustering-sentinel-intro"
---

# Introduction to Redis Sentinel

## Introduction

Redis Sentinel provides high availability for Redis without using Redis Cluster. It monitors your master-replica setup and automatically handles failover when the master fails, making it the standard HA solution for non-sharded Redis deployments.

## Key Concepts

- **Sentinel**: A separate process that monitors Redis instances and orchestrates failover
- **Quorum**: The minimum number of Sentinels that must agree a master is down before triggering failover
- **SDOWN (Subjectively Down)**: One Sentinel's individual opinion that a master is unreachable
- **ODOWN (Objectively Down)**: Consensus among quorum Sentinels that a master is truly down
- **Configuration Provider**: Clients query Sentinel to discover the current master address

## Real World Context

Without automated failover, a master failure requires manual intervention to promote a replica. Sentinel eliminates this human bottleneck, reducing downtime from minutes or hours to seconds. Most Redis deployments that do not need sharding use Sentinel for high availability.

## Deep Dive

### What Sentinel Does

1. **Monitoring**: Continuously checks if master and replicas are working
2. **Notification**: Alerts administrators via API when issues occur
3. **Automatic Failover**: Promotes a replica to master if master fails
4. **Configuration Provider**: Clients query Sentinel to find the current master

### Architecture

A typical Sentinel deployment runs 3 or 5 Sentinel processes alongside a Redis master and its replicas. Each Sentinel independently monitors all Redis instances and communicates with other Sentinels to reach consensus on the cluster state.

### Why Multiple Sentinels?

- **Consensus**: Multiple Sentinels must agree before failover (prevents split-brain)
- **Availability**: System works even if some Sentinels fail
- **Minimum 3**: Use odd number for majority voting (typically 3 or 5)

### Failover Process

1. Master becomes unreachable
2. Sentinels detect failure (SDOWN - Subjectively Down)
3. Multiple Sentinels confirm (ODOWN - Objectively Down)
4. Sentinels elect a leader Sentinel
5. Leader selects best replica for promotion
6. Replica is promoted to master
7. Other replicas reconfigure to follow new master
8. Clients are notified of new master

### Key Configuration Parameters

```bash
# How many Sentinels must agree master is down
sentinel monitor mymaster 192.168.1.10 6379 2
# "2" = quorum (minimum votes needed)

# Time before marking master as SDOWN
sentinel down-after-milliseconds mymaster 5000

# Failover timeout
sentinel failover-timeout mymaster 60000

# How many replicas can sync with new master simultaneously
sentinel parallel-syncs mymaster 1
```

### Quorum Explained

With 3 Sentinels and quorum = 2, failover proceeds when at least 2 Sentinels agree the master is down. This prevents a single Sentinel with network issues from triggering unnecessary failovers.

### SDOWN vs ODOWN

- **SDOWN (Subjectively Down)**: One Sentinel thinks master is down
- **ODOWN (Objectively Down)**: Quorum Sentinels agree master is down

Only ODOWN triggers failover.

## Common Pitfalls

1. **Deploying only 2 Sentinels** -- With 2 Sentinels and quorum of 2, losing one Sentinel prevents any failover. Always deploy at least 3 Sentinels.
2. **Placing all Sentinels in the same availability zone** -- If that zone goes down, you lose both the master and all Sentinels. Spread Sentinels across failure domains.

## Best Practices

1. **Use an odd number of Sentinels (3 or 5)** -- This ensures a clear majority for voting and prevents tie situations during leader election.
2. **Set quorum to majority** -- For 3 Sentinels use quorum 2, for 5 use quorum 3. This balances failure detection speed with split-brain prevention.

## Summary

- Sentinel provides automated failover for non-sharded Redis
- It uses a quorum-based consensus model to avoid false positives
- SDOWN is one Sentinel's opinion; ODOWN is consensus that triggers failover
- Deploy at least 3 Sentinels across different failure domains
- Clients connect through Sentinel to discover the current master

## Code Examples

**Essential Sentinel configuration parameters for monitoring a Redis master**

```bash
# Core sentinel configuration
sentinel monitor mymaster 192.168.1.10 6379 2
sentinel down-after-milliseconds mymaster 5000
sentinel failover-timeout mymaster 60000
sentinel parallel-syncs mymaster 1
```


## Resources

- [Redis Sentinel](https://redis.io/docs/latest/operate/oss_and_stack/management/sentinel/) — Complete Redis Sentinel guide

---

> 📘 *This lesson is part of the [Clustering and High Availability Architecture](https://stanza.dev/courses/redis-clustering) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*