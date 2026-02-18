---
source_course: "redis-messaging"
source_lesson: "redis-messaging-pubsub-intro"
---

# Introduction to Pub/Sub

Redis Pub/Sub (Publish/Subscribe) is a messaging paradigm where senders (publishers) don't send messages directly to receivers. Instead, messages are published to channels, and subscribers receive messages from channels they're interested in.

## How Pub/Sub Works

```
┌─────────────────────────────────────────────────────────────┐
│                     Redis Server                            │
│                                                             │
│  Publisher A ──PUBLISH──▶ [channel:news] ──▶ Subscriber 1  │
│                                          ──▶ Subscriber 2  │
│  Publisher B ──PUBLISH──▶ [channel:sports]──▶ Subscriber 3 │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**Key Concepts:**
- **Publisher**: Sends messages to a channel
- **Channel**: Named message conduit
- **Subscriber**: Listens to one or more channels
- **Message**: Data sent through channels

## Fire-and-Forget Model

Pub/Sub is **fire-and-forget**:
- Messages are not persisted
- If no subscribers are listening, messages are lost
- Publishers don't know if anyone received the message

```
┌─────────────────────────────────────────────────────────────┐
│  Publisher sends to channel "news":                         │
│                                                             │
│  No subscribers? → Message is lost                         │
│  3 subscribers?  → All 3 receive the message               │
│  Subscriber joins after? → Misses previous messages        │
└─────────────────────────────────────────────────────────────┘
```

## Basic Commands

### SUBSCRIBE

Subscribe to one or more channels:

```redis
# Subscribe to a single channel
SUBSCRIBE news

# Subscribe to multiple channels
SUBSCRIBE news sports weather
```

Once subscribed, the client enters **subscribe mode** and can only:
- Subscribe to more channels
- Unsubscribe
- Receive messages
- PING/QUIT

### PUBLISH

Send a message to a channel:

```redis
PUBLISH news "Breaking: Redis 8 released!"
# Returns: number of subscribers who received the message
# (integer) 2
```

### UNSUBSCRIBE

```redis
# Unsubscribe from specific channels
UNSUBSCRIBE news sports

# Unsubscribe from all channels
UNSUBSCRIBE
```

## Testing Pub/Sub with redis-cli

Open two terminal windows:

```bash
# Terminal 1: Subscriber
$ redis-cli
127.0.0.1:6379> SUBSCRIBE news
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "news"
3) (integer) 1
```

```bash
# Terminal 2: Publisher
$ redis-cli
127.0.0.1:6379> PUBLISH news "Hello, subscribers!"
(integer) 1
```

Terminal 1 receives:
```
1) "message"
2) "news"
3) "Hello, subscribers!"
```

## Pub/Sub Use Cases

✅ **Good for:**
- Real-time notifications
- Chat applications
- Live updates (scores, prices)
- Event broadcasting
- Invalidating distributed caches

❌ **Not suitable for:**
- Reliable message delivery (use Streams)
- Message persistence (use Streams)
- Message acknowledgment (use Streams)
- Work queues (use Lists or Streams)

📖 [Redis Pub/Sub](https://redis.io/docs/latest/develop/pubsub/)

## Resources

- [Redis Pub/Sub](https://redis.io/docs/latest/develop/pubsub/) — Official Redis Pub/Sub documentation

---

> 📘 *This lesson is part of the [Real-Time Messaging and Event Streaming](https://stanza.dev/courses/redis-messaging) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*