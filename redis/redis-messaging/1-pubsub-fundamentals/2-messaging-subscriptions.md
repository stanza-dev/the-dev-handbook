---
source_course: "redis-messaging"
source_lesson: "redis-messaging-subscriptions"
---

# Managing Subscriptions

## Introduction

Redis offers two subscription strategies: exact channel subscriptions and pattern subscriptions. Understanding both lets you build flexible message routing.

## Key Concepts

- **SUBSCRIBE**: registers exact channel names; a client can subscribe to multiple channels at once
- **PSUBSCRIBE**: registers glob-style patterns that match many channels in one call
- **pmessage**: the message type delivered when a pattern subscription matches (includes the matched pattern)

| Wildcard | Meaning |
|---|---|
| `*` | Zero or more characters |
| `?` | Exactly one character |
| `[ae]` | One character from the set |

## Real World Context

Pattern subscriptions are useful when channel names are dynamic at runtime — for example, subscribing to all user-specific notification channels with `PSUBSCRIBE notifications:user:*` from a logging or monitoring service.

## Deep Dive

```bash
PSUBSCRIBE news.*
# Matches: news.sports, news.tech, news.weather

PSUBSCRIBE user.?.events
# Matches: user.A.events, user.1.events
```

Messages delivered via pattern match arrive as `pmessage` (not `message`), and include the matched pattern:

```
1) "pmessage"
2) "news.*"       <- the pattern that matched
3) "news.sports"  <- the actual channel
4) "Goal!"
```

Inspecting active subscriptions:

```bash
PUBSUB CHANNELS            # list all active channels with subscribers
PUBSUB CHANNELS news.*     # filter by pattern
PUBSUB NUMSUB news sports  # count subscribers per channel
PUBSUB NUMPAT              # count active pattern subscriptions
```

## Common Pitfalls

- **Overlapping patterns**: if two patterns match the same channel, the subscriber receives two `pmessage` events for a single publish
- **Performance at scale**: Redis checks every published message against every registered pattern; thousands of patterns create CPU overhead
- **Forgetting PUNSUBSCRIBE**: unsubscribing from patterns requires `PUNSUBSCRIBE`, not `UNSUBSCRIBE`

## Best Practices

- Prefer exact `SUBSCRIBE` when the channel set is known and bounded
- Use `PSUBSCRIBE` only when channels are truly dynamic and the pattern set is small
- Monitor `PUBSUB NUMPAT` in production to detect pattern subscription leaks

## Summary

Use `SUBSCRIBE` for precise channels and `PSUBSCRIBE` for dynamic routing based on naming conventions. Pattern subscriptions are powerful but add slight overhead — Redis must check every published message against every active pattern.

## Code Examples

**Pattern Subscriptions**

```bash
# Subscribe to all news sub-channels
redis-cli PSUBSCRIBE 'news.*'

# In another terminal:
redis-cli PUBLISH news.sports "Goal!"
# Subscriber receives:
# 1) "pmessage"
# 2) "news.*"
# 3) "news.sports"
# 4) "Goal!"

# Inspect active channels
redis-cli PUBSUB CHANNELS
redis-cli PUBSUB NUMSUB news.sports
```


## Resources

- [Redis Pub/Sub](https://redis.io/docs/latest/develop/interact/pubsub/) — Official Redis Pub/Sub documentation

---

> 📘 *This lesson is part of the [Real-Time Messaging and Event Streaming](https://stanza.dev/courses/redis-messaging) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*