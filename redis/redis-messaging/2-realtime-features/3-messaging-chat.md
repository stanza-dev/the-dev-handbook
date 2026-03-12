---
source_course: "redis-messaging"
source_lesson: "redis-messaging-chat"
---

# Building Chat with Pub/Sub

## Introduction

Chat rooms are a textbook Pub/Sub use case. Each room is a channel; sending a message means publishing; joining a room means subscribing.

## Key Concepts

- **Room channel**: `chat:room:{roomId}` — all messages for a room
- **DM channel**: `chat:dm:{lowerUserId}:{higherUserId}` — sort IDs so there is only one channel per pair
- **Presence tracking**: Pub/Sub has no subscriber list API; track presence with a separate Redis Set and TTL heartbeat

## Real World Context

Pub/Sub maps directly to real-time chat:
- Chat apps (Slack-style): each channel is a Pub/Sub channel
- Live customer support: agent and customer both subscribe to `chat:ticket:{id}`
- Gaming lobbies: players subscribe to `game:lobby:{id}` for coordination messages

## Deep Dive

```
Channel naming:
  chat:room:{roomId}          <- room messages
  chat:presence:{roomId}      <- join/leave events
  chat:dm:{userId1}:{userId2} <- direct messages
```

```bash
# User joins room 5
SUBSCRIBE chat:room:5

# User sends a message (from another connection or via REST API)
PUBLISH chat:room:5 '{"userId":42,"text":"Hello!","ts":1691234567}'
# => (integer) 3  <- 3 people in the room received it
```

For direct messages, sort IDs so the channel name is stable regardless of who sends:

```bash
# Sort IDs so dm:100:200 is always the same channel regardless of sender
PUBLISH chat:dm:100:200 '{"from":200,"text":"Hey!"}'
```

Presence tracking using a TTL-refreshed Set:

```bash
# On connect: add user to a Set with expiry
SADD presence:room:5 user:42
EXPIRE presence:room:5 30  # refresh every heartbeat

# Query who is in the room
SMEMBERS presence:room:5
```

## Common Pitfalls

- **No message history**: new joiners cannot see past messages — must be stored in a database or Stream
- **Unsorted DM channel IDs**: using `dm:200:100` and `dm:100:200` as different channels creates duplicates
- **Using SUBSCRIBE in a shared HTTP connection**: a subscribed connection is blocked; use a dedicated connection per subscriber

## Best Practices

- Always sort user IDs numerically when constructing DM channel names
- Persist messages to a database or Stream for history replay on rejoin
- Implement presence as a separate Redis Set with a heartbeat TTL, not through Pub/Sub

## Summary

Channel naming conventions map naturally to chat concepts (rooms, DMs, presence channels). Use sorted user IDs for DM channel names to avoid duplicates. Always pair Pub/Sub chat with a persistence layer for message history.

## Code Examples

**Chat room setup**

```bash
# Join a chat room
SUBSCRIBE chat:room:42

# Send a message
PUBLISH chat:room:42 '{"userId":7,"text":"Hi everyone"}'

# Direct message (user IDs sorted: 100 < 200)
SUBSCRIBE chat:dm:100:200
PUBLISH chat:dm:100:200 '{"from":200,"text":"Hey!"}'

# Presence: track online users
SADD presence:room:42 user:7
EXPIRE presence:room:42 30
SMEMBERS presence:room:42
```


## Resources

- [Redis Pub/Sub](https://redis.io/docs/latest/develop/interact/pubsub/) — Official Redis Pub/Sub documentation

---

> 📘 *This lesson is part of the [Real-Time Messaging and Event Streaming](https://stanza.dev/courses/redis-messaging) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*