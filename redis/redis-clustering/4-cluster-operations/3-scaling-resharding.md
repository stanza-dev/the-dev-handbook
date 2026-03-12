---
source_course: "redis-clustering"
source_lesson: "redis-clustering-scaling-resharding"
---

# Scaling and Resharding Operations

## Introduction

As your data grows, you need to add nodes to your Redis Cluster and redistribute hash slots. Resharding is the process of moving slots between nodes, and it can be done online without downtime.

## Key Concepts

- **Resharding**: Moving hash slots from one node to another while the cluster remains operational
- **Rebalancing**: Automatically redistributing slots evenly across all master nodes
- **Slot Migration**: The atomic process of moving a single slot, including all its keys, between nodes
- **IMPORTING/MIGRATING state**: Temporary slot states during migration that handle ASK redirects

## Real World Context

Redis Cluster scaling is an online operation -- you never need to take the cluster offline to add capacity. However, resharding under heavy write load requires careful planning to minimize performance impact.

## Deep Dive

### Adding a New Master Node

```bash
# 1. Start a new Redis instance in cluster mode
redis-server --port 7006 --cluster-enabled yes \
  --cluster-config-file nodes-7006.conf

# 2. Add it to the cluster (initially has 0 slots)
redis-cli --cluster add-node 127.0.0.1:7006 127.0.0.1:7000

# 3. Verify it joined
redis-cli -c -p 7000 CLUSTER NODES
```

### Adding a Replica

```bash
# Add node as replica of a specific master
redis-cli --cluster add-node 127.0.0.1:7007 127.0.0.1:7000 \
  --cluster-slave --cluster-master-id <master-node-id>
```

### Resharding Slots

```bash
# Move 1000 slots from one node to another
redis-cli --cluster reshard 127.0.0.1:7000 \
  --cluster-from <source-node-id> \
  --cluster-to <target-node-id> \
  --cluster-slots 1000 \
  --cluster-yes
```

### How Slot Migration Works

For each slot being migrated:
1. Target node is set to IMPORTING state for the slot
2. Source node is set to MIGRATING state for the slot
3. Keys in the slot are moved one by one using MIGRATE
4. During migration, source handles existing keys and sends ASK redirects for migrated keys
5. Once all keys are moved, slot ownership is updated cluster-wide

```redis
# Monitor migration progress
CLUSTER COUNTKEYSINSLOT <slot-number>

# Check which slots are in migration
CLUSTER NODES
# Look for [<slot>->-<target-id>] (migrating)
# and [<slot>-<-<source-id>] (importing)
```

### Rebalancing

```bash
# Automatically distribute slots evenly
redis-cli --cluster rebalance 127.0.0.1:7000

# Rebalance with weight (give node 2x slots)
redis-cli --cluster rebalance 127.0.0.1:7000 \
  --cluster-weight <node-id>=2.0

# Check current distribution
redis-cli --cluster info 127.0.0.1:7000
```

### Removing a Node

```bash
# 1. Reshard all slots away from the node
redis-cli --cluster reshard 127.0.0.1:7000 \
  --cluster-from <node-to-remove> \
  --cluster-to <another-node> \
  --cluster-slots <all-remaining-slots> \
  --cluster-yes

# 2. Remove the empty node
redis-cli --cluster del-node 127.0.0.1:7000 <node-to-remove-id>
```

### Performance Impact During Resharding

- **MIGRATE** blocks the source node briefly for each key batch
- Large keys (>1MB) cause longer blocking periods
- ASK redirects add one extra round-trip for affected keys
- Schedule resharding during low-traffic periods when possible

## Common Pitfalls

1. **Removing a node before resharding its slots** -- If you delete a node that still owns slots, those slots become uncovered and the cluster enters FAIL state. Always reshard first.
2. **Resharding during peak traffic** -- Large key migrations can cause brief latency spikes. Schedule resharding during maintenance windows or off-peak hours.

## Best Practices

1. **Monitor CLUSTER INFO during resharding** -- Watch `cluster_slots_ok` to confirm all slots remain covered throughout the process.
2. **Use --cluster-yes for automation** -- In scripts, add `--cluster-yes` to skip interactive confirmations, but always test the script manually first.

## Summary

- Add nodes with `--cluster add-node`, then reshard slots to them
- Resharding moves slots online without downtime using MIGRATE
- Rebalancing automatically distributes slots evenly across masters
- Always reshard slots away from a node before removing it
- Schedule resharding during low-traffic periods to minimize latency impact

## Code Examples

**Add a node and redistribute slots in a Redis Cluster**

```bash
# Add a new master node
redis-cli --cluster add-node 127.0.0.1:7006 127.0.0.1:7000

# Reshard 1000 slots to the new node
redis-cli --cluster reshard 127.0.0.1:7000 \
  --cluster-from <source-id> \
  --cluster-to <new-node-id> \
  --cluster-slots 1000 --cluster-yes

# Rebalance slots evenly
redis-cli --cluster rebalance 127.0.0.1:7000
```


## Resources

- [Redis Cluster Scaling](https://redis.io/docs/latest/operate/oss_and_stack/management/scaling/) — Official guide to scaling and resharding Redis Cluster

---

> 📘 *This lesson is part of the [Clustering and High Availability Architecture](https://stanza.dev/courses/redis-clustering) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*