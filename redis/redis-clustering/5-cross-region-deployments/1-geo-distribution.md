---
source_course: "redis-clustering"
source_lesson: "redis-clustering-geo-distribution"
---

# Geo-Distribution Concepts

## Introduction

Running Redis across multiple datacenters presents unique challenges and trade-offs. Geo-distribution enables disaster recovery, lower latency for global users, and compliance with data locality regulations.

## Key Concepts

- **Active-Passive**: One primary region handles all writes; standby regions have read-only replicas
- **Active-Active**: Multiple regions accept writes simultaneously, requiring conflict resolution
- **Split-Brain**: Scenario where a network partition causes two sides to independently accept writes
- **CRDTs**: Conflict-free Replicated Data Types used by Redis Enterprise for Active-Active conflict resolution

## Real World Context

Global applications need Redis close to users for low latency. But cross-region replication adds complexity: you must choose between consistency (Active-Passive) and availability (Active-Active). Most teams start with Active-Passive and move to Active-Active only when latency requirements demand it.

## Deep Dive

### Why Multi-Region?

- Disaster Recovery: Survive datacenter failures
- Lower Latency: Serve users from nearest region
- Data Locality: Meet compliance requirements
- High Availability: 99.99%+ uptime

### Active-Passive (Primary-Standby)

```
US-East (Primary)          EU-West (Standby)
+-----------------+        +-----------------+
|  Redis Master   |------->|  Redis Replica  |
|  (Read/Write)   | Async  |  (Read-only)    |
+-----------------+  Repl  +-----------------+
```

```redis
# EU-West replica configuration
replicaof us-east-master.example.com 6379
replica-read-only yes
```

**Trade-offs:**
- Simple setup, no write conflicts
- Higher latency for distant users writing
- Manual failover required

### Active-Active (Multi-Primary)

Both regions accept writes with bidirectional sync. Requires conflict resolution (CRDTs in Redis Enterprise) or custom implementation.

**Trade-offs:**
- Low latency writes in all regions
- More complex operational model
- Conflict resolution needed

### Network Partitioning (Split-Brain)

During a network partition, both sides may think they are the primary and accept conflicting writes.

**Mitigation strategies:**
- Sentinel quorum across regions
- Minimum replica requirement before accepting writes
- Conflict-free data structures (CRDTs)

```redis
# Require at least 1 replica to accept writes
min-replicas-to-write 1
min-replicas-max-lag 10
```

### Latency Considerations

```
Typical Inter-Region Latencies:
US-East <-> US-West:    ~60-70ms
US-East <-> EU-West:    ~70-90ms
US-East <-> AP-South:   ~200-250ms

Synchronous replication: Adds full round-trip
Asynchronous replication: Risk of data loss
```

## Common Pitfalls

1. **Synchronous replication across regions** -- Adding 70-90ms to every write for cross-Atlantic sync is usually unacceptable. Use asynchronous replication and accept the RPO trade-off.
2. **Not testing failover to standby region** -- Many teams discover their standby is misconfigured only during a real disaster. Test region failover quarterly.

## Best Practices

1. **Start with Active-Passive** -- It is simpler, avoids conflict resolution, and works for most applications. Only move to Active-Active when write latency in remote regions is a proven problem.
2. **Use min-replicas-to-write** -- This prevents the master from accepting writes when no replicas are connected, reducing split-brain risk.

## Summary

- Active-Passive is simpler and sufficient for most use cases
- Active-Active enables low-latency writes everywhere but needs conflict resolution
- Use `min-replicas-to-write` to mitigate split-brain scenarios
- Cross-region latency (60-250ms) makes synchronous replication impractical
- Test disaster recovery failover procedures regularly

## Code Examples

**Cross-region replica configuration with split-brain protection**

```bash
# Configure replica for cross-region setup
replicaof us-east-master.example.com 6379
replica-read-only yes

# Prevent writes when no replicas connected (split-brain protection)
min-replicas-to-write 1
min-replicas-max-lag 10
```


## Resources

- [Active-Active Geo-Distribution](https://redis.io/docs/latest/operate/rs/databases/active-active/) — Redis Enterprise Active-Active documentation

---

> 📘 *This lesson is part of the [Clustering and High Availability Architecture](https://stanza.dev/courses/redis-clustering) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*