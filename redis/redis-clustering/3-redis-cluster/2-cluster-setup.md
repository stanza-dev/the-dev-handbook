---
source_course: "redis-clustering"
source_lesson: "redis-clustering-cluster-setup"
---

# Setting Up Redis Cluster

Let's create a 6-node cluster (3 masters, 3 replicas).

## Node Configuration

Each node needs cluster mode enabled:

```bash
# redis-cluster-node.conf
port 7000
cluster-enabled yes
cluster-config-file nodes-7000.conf
cluster-node-timeout 5000
appendonly yes

# For production
bind 0.0.0.0
requirepass your_password
masterauth your_password
```

## Starting Nodes

Start 6 instances (ports 7000-7005):

```bash
# Create directories
mkdir -p cluster/{7000,7001,7002,7003,7004,7005}

# Start each node
redis-server cluster/7000/redis.conf
redis-server cluster/7001/redis.conf
# ... repeat for all 6
```

## Creating the Cluster

Use redis-cli to create the cluster:

```bash
# Create cluster with 3 masters (replicas assigned automatically)
redis-cli --cluster create \
  127.0.0.1:7000 \
  127.0.0.1:7001 \
  127.0.0.1:7002 \
  127.0.0.1:7003 \
  127.0.0.1:7004 \
  127.0.0.1:7005 \
  --cluster-replicas 1

# You'll see:
# >>> Performing hash slots allocation on 6 nodes...
# Master[0] -> Slots 0 - 5460
# Master[1] -> Slots 5461 - 10922
# Master[2] -> Slots 10923 - 16383
# Adding replica 127.0.0.1:7004 to 127.0.0.1:7000
# ...
# Can I set the above configuration? (type 'yes' to accept): yes
```

## Cluster Commands

```redis
# Connect to any node
redis-cli -c -p 7000

# Cluster info
CLUSTER INFO

# List all nodes
CLUSTER NODES

# Slot information
CLUSTER SLOTS

# Find key's slot
CLUSTER KEYSLOT mykey
```

## Using the Cluster

```redis
# Connect with -c flag for automatic redirect handling
redis-cli -c -p 7000

# Set a key (auto-redirects to correct node)
127.0.0.1:7000> SET foo bar
-> Redirected to slot [12182] located at 127.0.0.1:7002
OK

127.0.0.1:7002> GET foo
"bar"
```

## Adding Nodes

```bash
# Add a new node as master
redis-cli --cluster add-node \
  127.0.0.1:7006 \
  127.0.0.1:7000

# Add as replica of specific master
redis-cli --cluster add-node \
  127.0.0.1:7006 \
  127.0.0.1:7000 \
  --cluster-slave \
  --cluster-master-id <node-id>
```

## Rebalancing Slots

```bash
# Redistribute slots evenly
redis-cli --cluster rebalance 127.0.0.1:7000

# Reshard specific number of slots
redis-cli --cluster reshard 127.0.0.1:7000 \
  --cluster-from <source-node-id> \
  --cluster-to <target-node-id> \
  --cluster-slots 1000
```

## Removing Nodes

```bash
# First reshard slots away from the node
redis-cli --cluster reshard 127.0.0.1:7000 \
  --cluster-from <node-to-remove-id> \
  --cluster-to <another-node-id> \
  --cluster-slots 5461 \
  --cluster-yes

# Then remove the empty node
redis-cli --cluster del-node \
  127.0.0.1:7000 \
  <node-to-remove-id>
```

## Client Connection

```python
from redis.cluster import RedisCluster

# Connect to cluster
rc = RedisCluster(
    host='127.0.0.1',
    port=7000,
    password='your_password'
)

# Use normally - client handles routing
rc.set('key', 'value')
value = rc.get('key')

# Multi-key with hash tags
rc.mset({'{user:1}:name': 'Alice', '{user:1}:age': '30'})
rc.mget('{user:1}:name', '{user:1}:age')
```

📖 [Create Redis Cluster](https://redis.io/docs/latest/operate/oss_and_stack/management/scaling/)

---

> 📘 *This lesson is part of the [Clustering and High Availability Architecture](https://stanza.dev/courses/redis-clustering) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*