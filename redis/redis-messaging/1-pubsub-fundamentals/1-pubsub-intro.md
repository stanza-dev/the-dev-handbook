---
source_course: "redis-messaging"
source_lesson: "redis-messaging-pubsub-intro"
---

# Introduction to Pub/Sub

## Introduction

Redis Pub/Sub (Publish/Subscribe) is a messaging paradigm where senders (publishers) don't send messages directly to receivers. Instead, messages are published to named channels, and any client that has subscribed to those channels receives them.

## Key Concepts

The three actors in a Pub/Sub system are the publisher, the channel, and the subscriber:

- **Publisher**: Sends a message to a named channel using `PUBLISH`
- **Channel**: A named conduit inside Redis — not a persistent data structure
- **Subscriber**: Registers interest in a channel using `SUBSCRIBE`; receives messages in real time

```
Publisher ──PUBLISH channel msg──▶ [Redis channel] ──▶ Subscriber 1
                                                   ──▶ Subscriber 2
```

## Real World Context

Pub/Sub is ideal for live, ephemeral broadcast scenarios where speed matters more than durability:

- Live score updates in a sports app
- Real-time price tickers in a trading dashboard
- Chat room broadcasts
- Live notifications pushed to connected browsers

## Deep Dive

```bash
# Subscribe to a channel (blocks the connection)
SUBSCRIBE news
# => 1) "subscribe"
# => 2) "news"
# => 3) (integer) 1

# In another client, publish a message
PUBLISH news "Hello subscribers!"
# => (integer) 2   <- number of subscribers who received it

# Subscriber receives:
# 1) "message"
# 2) "news"
# 3) "Hello subscribers!"
```

A subscribed client cannot issue regular commands — it enters a dedicated Pub/Sub mode where only `SUBSCRIBE`, `UNSUBSCRIBE`, `PSUBSCRIBE`, `PUNSUBSCRIBE`, `PING`, and `RESET` are allowed.

## Common Pitfalls

- **Assuming messages persist**: Pub/Sub has no storage — if no subscriber is connected, the message is gone
- **Using a subscribed connection for other commands**: the connection is locked in Pub/Sub mode until you unsubscribe
- **Missing messages between subscribe and first message**: there is a race window; subscribe before publishing if ordering matters

## Best Practices

- Use descriptive channel namespaces (e.g., `notifications:user:42`) to avoid collisions
- Keep message payloads small; use a key or ID and let consumers fetch data from Redis or a database
- Pair Pub/Sub with a persistent store for any use case where missing a message is not acceptable

## Summary

Pub/Sub decouples message producers from consumers through named channels. Publishers fire messages without knowing who is listening, and subscribers receive messages without knowing who sent them. This decoupling makes it easy to add new consumers without changing publisher code.

## Code Examples

**Basic Pub/Sub**

```bash
# Terminal 1 — Subscriber
redis-cli
> SUBSCRIBE news sports
# Blocks, waiting for messages
# 1) "subscribe"
# 2) "news"
# 3) (integer) 1

# Terminal 2 — Publisher
redis-cli
> PUBLISH news "Breaking: Redis 8 released!"
# (integer) 1   <- 1 subscriber received it

# Terminal 1 receives:
# 1) "message"
# 2) "news"
# 3) "Breaking: Redis 8 released!"
```


## Resources

- [Redis Pub/Sub](https://redis.io/docs/latest/develop/interact/pubsub/) — Official Redis Pub/Sub documentation

---

> 📘 *This lesson is part of the [Real-Time Messaging and Event Streaming](https://stanza.dev/courses/redis-messaging) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*