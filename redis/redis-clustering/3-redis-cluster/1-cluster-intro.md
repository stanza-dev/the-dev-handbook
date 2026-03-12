---
source_course: "redis-clustering"
source_lesson: "redis-clustering-cluster-intro"
---

# Redis Cluster Architecture

## Introduction

Redis Cluster provides automatic data sharding across multiple nodes with built-in high availability. Unlike Sentinel, which manages a single dataset with failover, Cluster distributes data across multiple masters for horizontal scaling.

## Key Concepts

- **Hash Slots**: Redis Cluster divides the keyspace into 16384 hash slots distributed across master nodes
- **Hash Tags**: Curly-brace syntax `{tag}` that forces related keys to the same slot
- **MOVED Redirect**: Response telling the client a key lives on a different node (permanent)
- **ASK Redirect**: Response during slot migration telling the client to temporarily try another node

## Real World Context

When your dataset outgrows a single server's memory or your write throughput exceeds what one master can handle, Redis Cluster lets you scale horizontally. It is the standard solution for large-scale Redis deployments.

## Deep Dive

### Hash Slots

Redis Cluster uses hash slots to distribute data:

```
16384 Hash Slots (0 - 16383)

Node A: Slots 0 - 5460
Node B: Slots 5461 - 10922
Node C: Slots 10923 - 16383

Key -> CRC16(key) % 16384 -> Slot -> Node
```

### Slot Calculation

```redis
# Find which slot a key belongs to
CLUSTER KEYSLOT mykey
# Returns: 14687

CLUSTER KEYSLOT user:1001
# Returns: 5244
```

### Hash Tags

Force related keys to the same slot:

```redis
# Keys with same {tag} go to same slot
{user:1001}:profile
{user:1001}:sessions
{user:1001}:orders

# Only content in {} is hashed
CLUSTER KEYSLOT {user:1001}:profile  # Same slot
CLUSTER KEYSLOT {user:1001}:orders   # Same slot
```

### Cluster Topology

Minimum: 3 Master Nodes. Recommended: 3 Masters + 3 Replicas (6 nodes). Each master handles a subset of hash slots and has one or more replicas for failover.

### Cluster vs Sentinel

| Feature | Sentinel | Cluster |
|---------|----------|---------|
| Sharding | No | Yes (automatic) |
| Max Data | 1 server's RAM | Sum of all masters |
| HA | Yes (failover) | Yes (per-shard) |
| Multi-key ops | Yes (all keys on same server) | Only with hash tags |
| Setup complexity | Medium | Higher |

### Request Routing

**Smart Clients** cache slot mappings and send requests directly to the correct node.

**MOVED Redirect** (permanent):
```redis
SET key value
(error) MOVED 14687 192.168.1.12:6379
# Client updates slot mapping and retries to correct node
```

**ASK Redirect** (during migration):
```redis
(error) ASK 14687 192.168.1.13:6379
# Client sends ASKING to target node, then retries (one-time)
```

### Cluster Limitations

1. **Multi-key operations** require all keys in same slot:
   ```redis
   # Works (same hash tag)
   MGET {user:1}:name {user:1}:email

   # Fails (different slots)
   MGET user:1:name user:2:name
   # (error) CROSSSLOT Keys in request don't hash to same slot
   ```
2. **Transactions** limited to single slot
3. **SELECT database** - Only database 0 is supported
4. **Lua scripts** must access only keys in same slot

## Common Pitfalls

1. **Forgetting hash tags for related keys** -- Without hash tags, related keys (e.g., user profile and user sessions) end up on different nodes, making multi-key operations impossible.
2. **Using non-cluster-aware clients** -- Basic Redis clients do not handle MOVED redirects. Always use a cluster-aware client library.

## Best Practices

1. **Design key naming with hash tags from the start** -- Retrofitting hash tags onto an existing application is painful. Plan your key naming strategy before deploying to Cluster.
2. **Use smart clients** -- Smart clients cache slot-to-node mappings and send requests directly, avoiding redirect overhead.

## Summary

- Redis Cluster shards data across masters using 16384 hash slots
- Hash tags ensure related keys land on the same node
- MOVED redirects are permanent; ASK redirects are temporary during migration
- Multi-key operations only work within a single slot
- Use cluster-aware client libraries for automatic redirect handling

## Code Examples

**Basic Redis Cluster inspection commands**

```bash
# Find which slot a key belongs to
redis-cli -c -p 7000 CLUSTER KEYSLOT mykey

# Check cluster status
redis-cli -c -p 7000 CLUSTER INFO

# List all nodes
redis-cli -c -p 7000 CLUSTER NODES
```


## Resources

- [Redis Cluster Specification](https://redis.io/docs/latest/operate/oss_and_stack/reference/cluster-spec/) — Technical specification of Redis Cluster

---

> 📘 *This lesson is part of the [Clustering and High Availability Architecture](https://stanza.dev/courses/redis-clustering) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*