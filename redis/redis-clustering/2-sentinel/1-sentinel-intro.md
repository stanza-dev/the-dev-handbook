---
source_course: "redis-clustering"
source_lesson: "redis-clustering-sentinel-intro"
---

# Introduction to Redis Sentinel

Redis Sentinel provides high availability for Redis without using Redis Cluster. It handles automatic failover when the master fails.

## What Sentinel Does

1. **Monitoring**: Continuously checks if master and replicas are working
2. **Notification**: Alerts administrators via API when issues occur  
3. **Automatic Failover**: Promotes a replica to master if master fails
4. **Configuration Provider**: Clients query Sentinel to find the current master

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Sentinel Cluster                         │
│   ┌──────────┐    ┌──────────┐    ┌──────────┐            │
│   │Sentinel 1│    │Sentinel 2│    │Sentinel 3│            │
│   └─────┬────┘    └─────┬────┘    └─────┬────┘            │
│         │               │               │                  │
│         └───────────────┼───────────────┘                  │
│                         │                                  │
│                    ┌────▼────┐                            │
│                    │  MASTER │                            │
│                    └────┬────┘                            │
│              ┌──────────┼──────────┐                      │
│         ┌────▼────┐          ┌────▼────┐                  │
│         │REPLICA 1│          │REPLICA 2│                  │
│         └─────────┘          └─────────┘                  │
└─────────────────────────────────────────────────────────────┘
```

## Why Multiple Sentinels?

- **Consensus**: Multiple Sentinels must agree before failover (prevents split-brain)
- **Availability**: System works even if some Sentinels fail
- **Minimum 3**: Use odd number for majority voting (typically 3 or 5)

## Failover Process

```
┌─────────────────────────────────────────────────────────────┐
│  1. Master becomes unreachable                              │
│  2. Sentinels detect failure (SDOWN - Subjectively Down)   │
│  3. Multiple Sentinels confirm (ODOWN - Objectively Down)  │
│  4. Sentinels elect a leader Sentinel                      │
│  5. Leader selects best replica for promotion              │
│  6. Replica is promoted to master                          │
│  7. Other replicas reconfigure to follow new master        │
│  8. Clients are notified of new master                     │
└─────────────────────────────────────────────────────────────┘
```

## Key Configuration Parameters

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

## Quorum Explained

```
┌─────────────────────────────────────────────────────────────┐
│  3 Sentinels, Quorum = 2                                    │
│                                                             │
│  Sentinel 1: "Master is down"     ✓                        │
│  Sentinel 2: "Master is down"     ✓  → Quorum reached!    │
│  Sentinel 3: "Master is up"       ✗                        │
│                                                             │
│  Failover will proceed because 2 >= quorum (2)             │
└─────────────────────────────────────────────────────────────┘
```

## SDOWN vs ODOWN

- **SDOWN (Subjectively Down)**: One Sentinel thinks master is down
- **ODOWN (Objectively Down)**: Quorum Sentinels agree master is down

Only ODOWN triggers failover.

📖 [Redis Sentinel](https://redis.io/docs/latest/operate/oss_and_stack/management/sentinel/)

## Resources

- [Redis Sentinel](https://redis.io/docs/latest/operate/oss_and_stack/management/sentinel/) — Complete Redis Sentinel guide

---

> 📘 *This lesson is part of the [Clustering and High Availability Architecture](https://stanza.dev/courses/redis-clustering) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*