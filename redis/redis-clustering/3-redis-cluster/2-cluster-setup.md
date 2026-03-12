---
source_course: "redis-clustering"
source_lesson: "redis-clustering-cluster-setup"
---

# Setting Up Redis Cluster

## Introduction

Creating a Redis Cluster involves starting multiple Redis instances in cluster mode, then using `redis-cli --cluster create` to form them into a cluster. This lesson walks through a complete 6-node setup.

## Key Concepts

- **cluster-enabled**: Redis config flag that enables cluster mode on a node
- **cluster-config-file**: Auto-managed file where each node stores cluster state
- **cluster-node-timeout**: How long a node must be unreachable before being considered failed
- **--cluster-replicas**: Number of replicas per master when creating the cluster

## Real World Context

Setting up a cluster correctly from the beginning avoids painful re-sharding later. Understanding the create, add-node, rebalance, and del-node commands gives you full control over your cluster topology.

## Deep Dive

### Node Configuration

Each node needs cluster mode enabled:

```bash
# redis-cluster-node.conf
port 7000
cluster-enabled yes
cluster-config-file nodes-7000.conf
cluster-node-timeout 5000
appendonly yes
bind 0.0.0.0
requirepass your_password
masterauth your_password
```

### Starting Nodes

Start 6 instances (ports 7000-7005):

```bash
mkdir -p cluster/{7000,7001,7002,7003,7004,7005}
redis-server cluster/7000/redis.conf
redis-server cluster/7001/redis.conf
# ... repeat for all 6
```

### Creating the Cluster

```bash
redis-cli --cluster create \
  127.0.0.1:7000 127.0.0.1:7001 127.0.0.1:7002 \
  127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005 \
  --cluster-replicas 1
```

### Cluster Commands

```redis
redis-cli -c -p 7000
CLUSTER INFO
CLUSTER NODES
CLUSTER SLOTS
CLUSTER KEYSLOT mykey
```

### Using the Cluster

```redis
# Connect with -c flag for automatic redirect handling
redis-cli -c -p 7000

127.0.0.1:7000> SET foo bar
-> Redirected to slot [12182] located at 127.0.0.1:7002
OK
```

### Adding Nodes

```bash
# Add a new node as master
redis-cli --cluster add-node 127.0.0.1:7006 127.0.0.1:7000

# Add as replica of specific master
redis-cli --cluster add-node 127.0.0.1:7006 127.0.0.1:7000 \
  --cluster-slave --cluster-master-id <node-id>
```

### Rebalancing Slots

```bash
redis-cli --cluster rebalance 127.0.0.1:7000

redis-cli --cluster reshard 127.0.0.1:7000 \
  --cluster-from <source-node-id> \
  --cluster-to <target-node-id> \
  --cluster-slots 1000
```

### Removing Nodes

```bash
# First reshard slots away from the node
redis-cli --cluster reshard 127.0.0.1:7000 \
  --cluster-from <node-to-remove-id> \
  --cluster-to <another-node-id> \
  --cluster-slots 5461 --cluster-yes

# Then remove the empty node
redis-cli --cluster del-node 127.0.0.1:7000 <node-to-remove-id>
```

### Client Connection

```python
from redis.cluster import RedisCluster

rc = RedisCluster(
    host='127.0.0.1',
    port=7000,
    password='your_password'
)

rc.set('key', 'value')
value = rc.get('key')

# Multi-key with hash tags
rc.mset({'{user:1}:name': 'Alice', '{user:1}:age': '30'})
rc.mget('{user:1}:name', '{user:1}:age')
```

## Common Pitfalls

1. **Forgetting masterauth in cluster mode** -- In a cluster, any node may become a replica after failover. If `masterauth` is not set, the node cannot authenticate with its new master and replication breaks.
2. **Starting cluster create with wrong node count** -- You need at least 6 nodes (3 masters + 3 replicas) for a production cluster. Running with 3 nodes means no replicas and no failover capability.

## Best Practices

1. **Always use the -c flag with redis-cli** -- Without `-c`, the CLI will not follow redirects and you will see MOVED errors instead of results.
2. **Start with 6 nodes minimum** -- 3 masters with 1 replica each provides both sharding and high availability.

## Summary

- Enable cluster mode with `cluster-enabled yes` in each node's config
- Use `redis-cli --cluster create` to form nodes into a cluster
- Add, remove, and rebalance nodes with `--cluster` subcommands
- Use cluster-aware client libraries (RedisCluster in Python) for applications
- Always set `masterauth` so nodes can authenticate after failover role changes

## Code Examples

**Create a Redis Cluster with 3 masters and 3 replicas**

```bash
# Create a 6-node cluster with 1 replica per master
redis-cli --cluster create \
  127.0.0.1:7000 127.0.0.1:7001 127.0.0.1:7002 \
  127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005 \
  --cluster-replicas 1
```


## Resources

- [Create Redis Cluster](https://redis.io/docs/latest/operate/oss_and_stack/management/scaling/) — Official guide to creating and scaling Redis Cluster

---

> 📘 *This lesson is part of the [Clustering and High Availability Architecture](https://stanza.dev/courses/redis-clustering) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*