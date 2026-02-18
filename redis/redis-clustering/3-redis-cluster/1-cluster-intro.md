---
source_course: "redis-clustering"
source_lesson: "redis-clustering-cluster-intro"
---

# Redis Cluster Architecture

Redis Cluster provides automatic data sharding across multiple nodes with built-in high availability.

## Key Concepts

### Hash Slots

Redis Cluster uses hash slots to distribute data:

```
┌─────────────────────────────────────────────────────────────┐
│  16384 Hash Slots (0 - 16383)                               │
│                                                             │
│  Node A: Slots 0 - 5460                                    │
│  Node B: Slots 5461 - 10922                                │
│  Node C: Slots 10923 - 16383                               │
│                                                             │
│  Key → CRC16(key) % 16384 → Slot → Node                    │
└─────────────────────────────────────────────────────────────┘
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

## Cluster Topology

```
┌─────────────────────────────────────────────────────────────┐
│  Minimum: 3 Master Nodes                                    │
│  Recommended: 3 Masters + 3 Replicas (6 nodes)             │
│                                                             │
│    ┌────────────┐    ┌────────────┐    ┌────────────┐      │
│    │  Master A  │    │  Master B  │    │  Master C  │      │
│    │ Slots 0-5k │    │Slots 5k-10k│    │Slots 10k+  │      │
│    └─────┬──────┘    └─────┬──────┘    └─────┬──────┘      │
│          │                 │                 │              │
│    ┌─────▼──────┐    ┌─────▼──────┐    ┌─────▼──────┐      │
│    │ Replica A' │    │ Replica B' │    │ Replica C' │      │
│    └────────────┘    └────────────┘    └────────────┘      │
└─────────────────────────────────────────────────────────────┘
```

## Cluster vs Sentinel

| Feature | Sentinel | Cluster |
|---------|----------|---------|
| Sharding | No | Yes (automatic) |
| Max Data | 1 server's RAM | Sum of all masters |
| HA | Yes (failover) | Yes (per-shard) |
| Multi-key ops | Yes (all keys on same server) | Only with hash tags |
| Setup complexity | Medium | Higher |

## Request Routing

### Smart Clients

Most clients cache slot mappings:

```
1. Client knows slot → node mapping
2. Client sends request directly to correct node
3. Fast, single hop
```

### MOVED Redirect

When client hits wrong node:

```redis
# Client sends to wrong node
SET key value
# Response:
(error) MOVED 14687 192.168.1.12:6379

# Client should:
# 1. Update slot mapping
# 2. Retry to correct node
```

### ASK Redirect

During slot migration:

```redis
(error) ASK 14687 192.168.1.13:6379

# Client should:
# 1. Send ASKING to target node
# 2. Retry command (one-time redirect)
```

## Cluster Limitations

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

📖 [Redis Cluster Specification](https://redis.io/docs/latest/operate/oss_and_stack/reference/cluster-spec/)

## Resources

- [Redis Cluster Specification](https://redis.io/docs/latest/operate/oss_and_stack/reference/cluster-spec/) — Technical specification of Redis Cluster

---

> 📘 *This lesson is part of the [Clustering and High Availability Architecture](https://stanza.dev/courses/redis-clustering) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*