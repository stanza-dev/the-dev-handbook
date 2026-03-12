---
source_course: "redis-messaging"
source_lesson: "redis-messaging-streams-intro"
---

# Redis Streams Fundamentals

## Introduction

Redis Streams is a persistent, append-only log data structure. Unlike Pub/Sub, messages are stored on disk and can be replayed at any time. It is Redis's answer to Kafka-style event streaming.

## Key Concepts

- **Stream entry ID**: format `millisecond-sequence` (e.g., `1691234567890-0`); monotonically increasing, ensuring chronological order
- **XADD**: appends an entry; `*` auto-generates the ID
- **XREAD**: reads entries forward from a given ID; `$` means only new entries; `BLOCK` makes it wait for new entries
- **Append-only**: entries are not modified after creation — only added or deleted

## Real World Context

- **Order pipeline**: each order change is an entry; microservices read and process asynchronously
- **IoT telemetry**: sensor readings appended as they arrive; consumers process at their own pace
- **Audit trail**: every user action is an entry; compliance team replays the stream for audits

## Deep Dive

Every entry in a stream has a unique ID with format `millisecond-sequence`:

```
1691234567890-0   <- first entry at that millisecond
1691234567890-1   <- second entry at the same millisecond
```

```bash
XADD orders '*' status pending customerId 42 amount 99.95
# => "1691234567890-0"  <- auto-generated ID

# Read up to 10 entries from the beginning
XREAD COUNT 10 STREAMS orders 0

# Read only new entries (block until one arrives, max 5 seconds)
XREAD BLOCK 5000 STREAMS orders $
```

| | Pub/Sub | Streams |
|---|---|---|
| Persistence | No | Yes |
| Replay | No | Yes |
| Backpressure | None | Consumer groups |
| Offline consumers | Lose messages | Catch up on reconnect |

## Common Pitfalls

- **Using `$` in XREAD without BLOCK**: `XREAD STREAMS orders $` without BLOCK returns nothing immediately — you must use BLOCK or start from `0`
- **Forgetting that XREAD is not a consumer group**: without a consumer group, every reader sees all messages
- **Unbounded stream growth**: streams grow indefinitely without explicit trimming (XTRIM)

## Best Practices

- Use `*` for auto-generated IDs in all production use cases
- Always pair XADD with MAXLEN trimming in high-throughput streams
- Start consumers from `0` for full replay or from `$` for live-only consumption

## Summary

Redis Streams combine the low latency of Pub/Sub with the durability of a message queue. The append-only log with auto-generated time-based IDs makes it easy to read from any point in time and replay the full history.

## Code Examples

**Basic Stream operations**

```bash
# Append entries to a stream
XADD orders '*' status pending customerId 42
# => "1691234567890-0"

XADD orders '*' status shipped orderId ord-1
# => "1691234567891-0"

# Read all entries from beginning
XREAD COUNT 100 STREAMS orders 0

# Block and wait for new entries
XREAD BLOCK 0 STREAMS orders $
```


## Resources

- [Redis Streams](https://redis.io/docs/latest/develop/data-types/streams/) — Official Redis Streams data type documentation

---

> 📘 *This lesson is part of the [Real-Time Messaging and Event Streaming](https://stanza.dev/courses/redis-messaging) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*