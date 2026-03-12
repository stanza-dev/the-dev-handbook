---
source_course: "redis-clustering"
source_lesson: "redis-clustering-cluster-clients"
---

# Cluster Client Libraries and Multi-Key Operations

## Introduction

Working with Redis Cluster from application code requires understanding smart clients, hash tag strategies, and the limitations that sharding imposes on multi-key operations, pipelines, and Lua scripts.

## Key Concepts

- **Smart Client**: A client that caches the slot-to-node mapping and routes commands directly to the correct node
- **Hash Tag**: The `{...}` syntax that controls which part of the key is used for slot calculation
- **CROSSSLOT Error**: Error returned when a multi-key command references keys in different slots
- **Cluster Pipeline**: A pipeline that groups commands by target node and sends them in parallel

## Real World Context

Most Cluster problems in production stem from application-level issues: CROSSSLOT errors breaking transactions, pipelines silently degrading performance, or Lua scripts that worked on standalone Redis failing in Cluster. Understanding these constraints upfront prevents costly refactoring.

## Deep Dive

### Smart Client Architecture

A smart client maintains a local copy of the slot map:

1. On startup, calls `CLUSTER SLOTS` or `CLUSTER SHARDS` to build the map
2. For each command, computes `CRC16(key) % 16384` to find the slot
3. Looks up the slot in the local map to find the target node
4. Sends the command directly (no redirects needed)
5. On MOVED error, refreshes the slot map

```python
from redis.cluster import RedisCluster

# Smart client automatically handles slot routing
rc = RedisCluster(host='127.0.0.1', port=7000)

# These go to different nodes transparently
rc.set('user:1', 'Alice')    # Slot X -> Node A
rc.set('user:2', 'Bob')      # Slot Y -> Node B
```

### Hash Tags In Depth

Hash tags let you control slot assignment:

```redis
# Without hash tags - different slots
CLUSTER KEYSLOT user:1:profile   # Slot 12539
CLUSTER KEYSLOT user:1:orders    # Slot 9visually

# With hash tags - same slot
CLUSTER KEYSLOT {user:1}:profile  # Slot 5649
CLUSTER KEYSLOT {user:1}:orders   # Slot 5649
```

**Rules:**
- Only the first `{...}` is used
- Empty braces `{}` means the whole key is hashed
- Nested braces are not supported

**Common patterns:**
```redis
# Per-user data
{user:123}:profile
{user:123}:settings
{user:123}:sessions

# Per-tenant data
{tenant:acme}:config
{tenant:acme}:users
```

### Pipeline Limitations

In standalone Redis, a pipeline sends all commands to one server. In Cluster, the client must split the pipeline by target node:

```python
# Cluster-aware pipeline (redis-py handles this automatically)
pipe = rc.pipeline()
pipe.set('{user:1}:name', 'Alice')
pipe.set('{user:1}:age', '30')
pipe.set('{user:2}:name', 'Bob')    # Different node
results = pipe.execute()
# Internally: 2 commands to Node A, 1 command to Node B
```

**Performance impact:** Cross-node pipelines have higher latency than single-node pipelines because the client must wait for all nodes to respond.

### Lua Scripting Constraints

Lua scripts in Cluster can only access keys in a single slot:

```redis
# Works: all keys share hash tag
EVAL "return redis.call('GET', KEYS[1]) .. redis.call('GET', KEYS[2])" 2 {user:1}:name {user:1}:age

# Fails: keys in different slots
EVAL "return redis.call('GET', KEYS[1])" 1 user:1:name
# If the script also accessed user:2:name -> CROSSSLOT error
```

**Workaround:** Pass all keys via hash-tagged KEYS parameters and design Lua scripts to operate on a single entity.

### Transaction Constraints

MULTI/EXEC transactions only work when all watched/accessed keys are in the same slot:

```redis
# Works
WATCH {user:1}:balance
MULTI
DECRBY {user:1}:balance 100
INCRBY {user:1}:spent 100
EXEC
```

## Common Pitfalls

1. **Assuming pipelines are atomic in Cluster** -- Unlike standalone Redis, cluster pipelines are split across nodes and can partially fail. Always check individual results.
2. **Hot slots from poor hash tag design** -- If too many keys share the same hash tag (e.g., `{global}:counter`), one node handles disproportionate traffic. Distribute hash tags across multiple entities.

## Best Practices

1. **Use entity-level hash tags** -- Tag by user, tenant, or order ID rather than by category. This distributes load while keeping related data together.
2. **Test multi-key operations locally with cluster mode** -- Run a local 6-node cluster during development to catch CROSSSLOT errors before production.

## Summary

- Smart clients cache slot mappings and route commands directly to the correct node
- Hash tags control slot assignment; design them around entity boundaries
- Pipelines in cluster mode are split by node and are not atomic
- Lua scripts and transactions can only access keys within a single slot
- Test with cluster mode enabled during development to catch CROSSSLOT errors early

## Code Examples

**Using RedisCluster with hash tags and cluster-aware pipelines**

```python
from redis.cluster import RedisCluster

rc = RedisCluster(host='127.0.0.1', port=7000)

# Hash-tagged keys go to same slot
rc.mset({'{user:1}:name': 'Alice', '{user:1}:age': '30'})
result = rc.mget('{user:1}:name', '{user:1}:age')

# Cluster pipeline (auto-split by node)
pipe = rc.pipeline()
pipe.set('{user:1}:score', '100')
pipe.set('{user:2}:score', '200')
results = pipe.execute()
```


## Resources

- [Redis Cluster Key Distribution](https://redis.io/docs/latest/operate/oss_and_stack/reference/cluster-spec/) — Hash slot and key distribution specification

---

> 📘 *This lesson is part of the [Clustering and High Availability Architecture](https://stanza.dev/courses/redis-clustering) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*