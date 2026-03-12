---
source_course: "redis-clustering"
source_lesson: "redis-clustering-failover-ops"
---

# Cluster Failover and Recovery

## Introduction

Understanding and managing failover scenarios in Redis Cluster is critical for keeping your deployment healthy. This lesson covers automatic failover, manual failover for maintenance, and recovery procedures.

## Key Concepts

- **Automatic Failover**: The cluster automatically promotes a replica when a master fails
- **CLUSTER FAILOVER**: Graceful manual failover coordinated between master and replica
- **CLUSTER FAILOVER FORCE**: Failover when master is unreachable but majority of masters are available
- **CLUSTER FAILOVER TAKEOVER**: Emergency failover without consensus

## Real World Context

Planned maintenance (OS patches, hardware upgrades, Redis version updates) requires graceful failovers. Unplanned failures require understanding of automatic recovery. Knowing the difference between FAILOVER, FORCE, and TAKEOVER can mean the difference between a smooth operation and data loss.

## Deep Dive

### Automatic Failover

When a master fails:
1. Master A becomes unreachable
2. Other masters detect failure (cluster-node-timeout)
3. Replica A' is promoted to master
4. Cluster updates slot mappings
5. Clients receive MOVED redirects to new master

### Manual Failover

For maintenance or planned upgrades:

```redis
# On the replica that should become master:
CLUSTER FAILOVER

# Force failover (even if master is unavailable)
CLUSTER FAILOVER FORCE

# Takeover without master agreement
CLUSTER FAILOVER TAKEOVER
```

| Type | Use Case |
|------|----------|
| CLUSTER FAILOVER | Graceful, coordinated with master |
| CLUSTER FAILOVER FORCE | Master unreachable but majority available |
| CLUSTER FAILOVER TAKEOVER | Emergency, no consensus needed |

### Checking Cluster Health

```redis
CLUSTER INFO
# cluster_state:ok
# cluster_slots_assigned:16384
# cluster_slots_ok:16384
# cluster_slots_pfail:0
# cluster_slots_fail:0

# If cluster_state:fail, investigate:
CLUSTER NODES
# Look for nodes marked as 'fail' or 'pfail'
```

### Node States

- **master/slave**: Role
- **myself**: Current node
- **pfail**: Possibly failing (one node's view)
- **fail**: Confirmed failing (consensus)
- **handshake**: Joining cluster
- **noaddr**: No address known

### Recovering a Failed Node

```bash
# 1. Fix the issue (restart Redis, fix network, etc.)
redis-server /path/to/redis.conf

# 2. Node automatically rejoins as replica
#    (if its master slot was taken over)

# 3. Verify recovery
redis-cli -c -p 7000 CLUSTER NODES
```

### Handling Network Partitions

During a network split, the minority side stops accepting writes and returns CLUSTERDOWN. The majority side promotes replicas and continues operating. This prevents split-brain data conflicts.

**Cluster requires majority** to accept writes.

### Force Reset (Emergency)

```redis
# Reset a node completely (removes all data!)
CLUSTER RESET HARD

# Soft reset (keeps data)
CLUSTER RESET SOFT
```

## Common Pitfalls

1. **Using TAKEOVER when FAILOVER would suffice** -- TAKEOVER bypasses consensus and can cause data inconsistency. Use it only when the majority of masters are truly unavailable.
2. **Not waiting for sync before maintenance** -- Before a planned failover, ensure the replica is fully synchronized with the master to avoid data loss.

## Best Practices

1. **Always have replicas** -- Each master should have at least one replica for automatic failover capability.
2. **Test failovers regularly** -- Practice recovery procedures quarterly so your team is prepared for real outages.

## Summary

- Use CLUSTER FAILOVER for graceful planned maintenance
- FORCE and TAKEOVER are for emergency scenarios with increasing risk
- Monitor cluster_state and node flags to detect issues early
- Failed nodes automatically rejoin as replicas after recovery
- Network partitions trigger CLUSTERDOWN on the minority side

## Code Examples

**Cluster health inspection and manual failover commands**

```bash
# Check cluster health
redis-cli -c -p 7000 CLUSTER INFO

# List all nodes and their states
redis-cli -c -p 7000 CLUSTER NODES

# Graceful failover from replica
redis-cli -c -p 7003 CLUSTER FAILOVER
```


## Resources

- [Cluster Failover](https://redis.io/docs/latest/commands/cluster-failover/) — CLUSTER FAILOVER command reference

---

> 📘 *This lesson is part of the [Clustering and High Availability Architecture](https://stanza.dev/courses/redis-clustering) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*