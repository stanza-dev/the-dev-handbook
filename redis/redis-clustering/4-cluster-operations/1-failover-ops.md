---
source_course: "redis-clustering"
source_lesson: "redis-clustering-failover-ops"
---

# Cluster Failover and Recovery

Understanding and managing failover scenarios in Redis Cluster.

## Automatic Failover

When a master fails:

```
┌─────────────────────────────────────────────────────────────┐
│  1. Master A becomes unreachable                            │
│  2. Other masters detect failure (cluster-node-timeout)    │
│  3. Replica A' is promoted to master                       │
│  4. Cluster updates slot mappings                          │
│  5. Clients receive MOVED redirects to new master          │
└─────────────────────────────────────────────────────────────┘
```

## Manual Failover

For maintenance or planned upgrades:

```redis
# On the replica that should become master:
CLUSTER FAILOVER

# Force failover (even if master is unavailable)
CLUSTER FAILOVER FORCE

# Takeover without master agreement
CLUSTER FAILOVER TAKEOVER
```

### Failover Types

| Type | Use Case |
|------|----------|
| CLUSTER FAILOVER | Graceful, coordinated with master |
| CLUSTER FAILOVER FORCE | Master unreachable but majority available |
| CLUSTER FAILOVER TAKEOVER | Emergency, no consensus needed |

## Checking Cluster Health

```redis
# Overall cluster status
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

## Node States

- **master/slave**: Role
- **myself**: Current node
- **pfail**: Possibly failing (one node's view)
- **fail**: Confirmed failing (consensus)
- **handshake**: Joining cluster
- **noaddr**: No address known

## Recovering a Failed Node

```bash
# 1. Fix the issue (restart Redis, fix network, etc.)
redis-server /path/to/redis.conf

# 2. Node automatically rejoins as replica
#    (if its master slot was taken over)

# 3. Verify recovery
redis-cli -c -p 7000 CLUSTER NODES
```

## Handling Network Partitions

```
┌─────────────────────────────────────────────────────────────┐
│  Network Split (Partition):                                 │
│                                                             │
│  [Master A] [Replica B']    |    [Master B] [Replica A']   │
│         Minority            |         Majority             │
│                             |                               │
│  Minority side:             |  Majority side:              │
│  - Stops accepting writes   |  - Promotes Replica A'       │
│  - Returns CLUSTERDOWN      |  - Continues operating       │
└─────────────────────────────────────────────────────────────┘
```

**Cluster requires majority** to accept writes (prevents split-brain).

## Force Reset (Emergency)

```redis
# Reset a node completely (removes all data!)
CLUSTER RESET HARD

# Soft reset (keeps data)
CLUSTER RESET SOFT
```

## Best Practices

1. **Always have replicas**: Each master should have at least one replica
2. **Spread across racks**: Use replica-migration for rack awareness
3. **Monitor cluster state**: Alert on cluster_state:fail
4. **Test failovers**: Regularly practice recovery procedures
5. **Document recovery steps**: Have runbooks ready

📖 [Cluster Failover](https://redis.io/docs/latest/commands/cluster-failover/)

---

> 📘 *This lesson is part of the [Clustering and High Availability Architecture](https://stanza.dev/courses/redis-clustering) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*