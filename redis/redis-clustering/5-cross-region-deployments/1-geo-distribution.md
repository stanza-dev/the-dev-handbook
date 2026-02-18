---
source_course: "redis-clustering"
source_lesson: "redis-clustering-geo-distribution"
---

# Geo-Distribution Concepts

Running Redis across multiple datacenters presents unique challenges and trade-offs.

## Why Multi-Region?

```
┌─────────────────────────────────────────────────────────────┐
│  Benefits of Multi-Region Deployment                        │
│                                                             │
│  • Disaster Recovery: Survive datacenter failures          │
│  • Lower Latency: Serve users from nearest region          │
│  • Data Locality: Meet compliance requirements             │
│  • High Availability: 99.99%+ uptime                       │
└─────────────────────────────────────────────────────────────┘
```

## Deployment Patterns

### Active-Passive (Primary-Standby)

```
US-East (Primary)          EU-West (Standby)
┌─────────────────┐        ┌─────────────────┐
│  Redis Master   │──────▶│  Redis Replica  │
│  (Read/Write)   │  Async │  (Read-only)    │
└─────────────────┘  Repl  └─────────────────┘
```

```redis
# EU-West replica configuration
replicaof us-east-master.example.com 6379

# Enable read queries on replica (optional)
replica-read-only yes
```

**Trade-offs:**
- Simple setup
- No write conflicts
- Higher latency for distant users
- Manual failover required

### Active-Active (Multi-Primary)

```
US-East (Primary)          EU-West (Primary)
┌─────────────────┐        ┌─────────────────┐
│  Redis Master   │◀─────▶│  Redis Master   │
│  (Read/Write)   │ 2-way  │  (Read/Write)   │
└─────────────────┘  Sync  └─────────────────┘
```

**Requirements:**
- Conflict resolution mechanism (CRDTs)
- Redis Enterprise or custom implementation
- More complex operational model

## Network Partitioning (Split-Brain)

```
                  Network Partition
                        ║
US-East                 ║           EU-West
┌─────────────────┐     ║     ┌─────────────────┐
│  Redis Master   │◀────╳────▶│  Redis Master   │
│  (Thinks it's   │     ║     │  (Thinks it's   │
│   the leader)   │     ║     │   the leader)   │
└─────────────────┘     ║     └─────────────────┘
        │               ║             │
        ▼               ║             ▼
   Clients write        ║      Clients write
   conflicting data     ║      conflicting data
```

**Mitigation strategies:**
- Sentinel quorum across regions
- Minimum replica requirement before accepting writes
- Conflict-free data structures (CRDTs)

```redis
# Require at least 1 replica to accept writes
min-replicas-to-write 1
min-replicas-max-lag 10
```

## Latency Considerations

```
Typical Inter-Region Latencies:
─────────────────────────────────
US-East ↔ US-West:    ~60-70ms
US-East ↔ EU-West:    ~70-90ms
US-East ↔ AP-South:   ~200-250ms

Synchronous replication: Adds full round-trip
Asynchronous replication: Risk of data loss
```

📖 [Redis Enterprise Active-Active](https://redis.io/docs/latest/operate/rs/databases/active-active/)

## Resources

- [Active-Active Geo-Distribution](https://redis.io/docs/latest/operate/rs/databases/active-active/) — Redis Enterprise Active-Active documentation

---

> 📘 *This lesson is part of the [Clustering and High Availability Architecture](https://stanza.dev/courses/redis-clustering) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*