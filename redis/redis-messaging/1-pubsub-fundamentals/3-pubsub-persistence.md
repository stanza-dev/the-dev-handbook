---
source_course: "redis-messaging"
source_lesson: "redis-messaging-pubsub-persistence"
---

# Pub/Sub Message Persistence and Limitations

## Introduction

One of the most important characteristics of Redis Pub/Sub is what it does NOT do: it does not persist messages. Understanding this limitation is essential for choosing the right tool.

## Key Concepts

- **At-most-once delivery**: a message is delivered to currently connected subscribers and immediately discarded — no queue, no retry
- **Fire-and-forget**: the publisher does not know whether anyone received the message
- **Output buffer**: slow subscribers accumulate messages in a client output buffer; if it fills, the connection is dropped

## Real World Context

| Use Case | Pub/Sub? |
|---|---|
| Live dashboard refresh (missing one update is OK) | Yes |
| Chat notification ping (UI will poll if missed) | Yes |
| Order processing (every event must be handled) | No — use Streams |
| Audit log (must never lose data) | No — use Streams |

## Deep Dive

```bash
# Subscriber connects and subscribes
SUBSCRIBE orders

# --- subscriber crashes ---

# Publisher sends 100 messages while subscriber is down
PUBLISH orders "order:1"
# ... (99 more)

# Subscriber reconnects
SUBSCRIBE orders
# It sees ZERO of those 100 messages — they are gone
```

| Feature | Pub/Sub | Streams |
|---|---|---|
| Persistence | No | Yes |
| Replay | No | Yes (XRANGE, XREAD) |
| Consumer groups | No | Yes |
| Delivery guarantee | At-most-once | At-least-once |
| Latency | Very low | Low |

## Common Pitfalls

- **Assuming messages persist**: they don't — a restart loses everything
- **Using Pub/Sub for critical pipelines**: if a consumer restarts, it misses events
- **Infinite subscriber output buffers**: large burst traffic can overflow and disconnect slow consumers

## Best Practices

- Use Pub/Sub for **ephemeral notifications** (UI triggers, cache invalidation signals)
- Use Streams for **durable event logs** (order processing, audit trails, analytics)
- If you need both low latency and durability, publish to a Stream and let consumers react

## Summary

Pub/Sub is optimized for speed, not safety. Messages live only as long as there are connected subscribers to receive them. For anything requiring guaranteed delivery, replay, or offline consumer support, Redis Streams is the right choice.

## Code Examples

**Message loss demonstration**

```bash
# Subscriber connects
SUBSCRIBE important-events

# --- subscriber goes offline ---

# Messages published during downtime
PUBLISH important-events "event-1"   # nobody receives this
PUBLISH important-events "event-2"   # nobody receives this

# Subscriber reconnects
SUBSCRIBE important-events
# event-1 and event-2 are GONE — not delivered

# For durable delivery, use Streams instead:
XADD important-events '*' type event-1
XADD important-events '*' type event-2
# Consumer can replay from any point with XREAD or XRANGE
```


## Resources

- [Redis Pub/Sub](https://redis.io/docs/latest/develop/interact/pubsub/) — Official Redis Pub/Sub documentation

---

> 📘 *This lesson is part of the [Real-Time Messaging and Event Streaming](https://stanza.dev/courses/redis-messaging) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*