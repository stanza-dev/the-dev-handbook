---
source_course: "redis-messaging"
source_lesson: "redis-messaging-sharded-pubsub"
---

# Sharded Pub/Sub for Redis Cluster

## Introduction

Introduced in Redis 7.0, Sharded Pub/Sub solves a fundamental scalability problem with classic Pub/Sub in a Redis Cluster: every published message is broadcast to all cluster nodes, regardless of which shard the channel belongs to. This creates unnecessary cross-node traffic.

## Key Concepts

- **Channel pinning**: with Sharded Pub/Sub, each channel is pinned to the cluster shard that owns its hash slot — exactly like keys
- **SSUBSCRIBE / SPUBLISH / SUNSUBSCRIBE**: the S-prefixed commands for sharded subscriptions
- **Isolation**: sharded Pub/Sub subscribers cannot receive messages from classic `PUBLISH` and vice versa

## Real World Context

Large-scale real-time systems (millions of connected users) use Sharded Pub/Sub to scale horizontally without the broadcast overhead. Each microservice connects to the shard managing its domain channels.

## Deep Dive

In a Redis Cluster, when you call `PUBLISH channel message`, the message is propagated to every node in the cluster. If you have 10 nodes and 1000 channels, even a small publish storm floods all nodes.

With Sharded Pub/Sub, messages only travel within the owning shard:

```
Cluster: [Shard A] [Shard B] [Shard C]

SPUBLISH orders:new "ord-42"
  → only delivered within Shard A (where orders:new hashes to)
  → Shards B and C are not involved
```

```bash
# Subscribe to a sharded channel
SSUBSCRIBE orders:new           # sharded (stays in one shard)

# Publish to a sharded channel
SPUBLISH orders:new "ord-42"   # only reaches SSUBSCRIBE subscribers

# Unsubscribe
SUNSUBSCRIBE orders:new
```

`SPUBLISH` and `SSUBSCRIBE` are cluster-aware — you must connect to the correct shard node (or use a smart client). Channel naming follows the same hash slot rules as keys: `{tag}` can force co-location.

## Common Pitfalls

- **Mixing PUBLISH and SPUBLISH**: subscribers using `SSUBSCRIBE` will NOT receive messages sent via classic `PUBLISH` — they are separate systems
- **Connecting to the wrong shard**: `SPUBLISH` must be issued on the node owning the channel's hash slot; smart clients handle this automatically
- **Using classic Pub/Sub in a large cluster**: each PUBLISH floods all nodes, causing significant cross-shard traffic at scale

## Best Practices

- Migrate to Sharded Pub/Sub when operating Redis Cluster with more than a few nodes
- Use `{tag}` hash tags in channel names to co-locate related channels on the same shard
- Use a cluster-aware client library that auto-routes SPUBLISH to the correct shard

## Summary

Sharded Pub/Sub (Redis 7.0+) eliminates cluster-wide message broadcast by pinning channels to their natural shard. Use `SSUBSCRIBE`/`SPUBLISH`/`SUNSUBSCRIBE` instead of `SUBSCRIBE`/`PUBLISH` when operating in a Redis Cluster to reduce cross-shard traffic significantly.

## Code Examples

**Sharded Pub/Sub basics**

```bash
# Subscriber (must be connected to the correct shard)
SSUBSCRIBE orders:new
# 1) "ssubscribe"
# 2) "orders:new"
# 3) (integer) 1

# Publisher
SPUBLISH orders:new '{"orderId":"ord-42","status":"created"}'
# (integer) 1  <- only within this shard

# Unsubscribe
SUNSUBSCRIBE orders:new
```


## Resources

- [Sharded Pub/Sub](https://redis.io/docs/latest/develop/interact/pubsub/) — Official Redis Pub/Sub with sharding documentation

---

> 📘 *This lesson is part of the [Real-Time Messaging and Event Streaming](https://stanza.dev/courses/redis-messaging) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*