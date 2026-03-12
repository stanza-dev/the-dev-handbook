---
source_course: "redis-messaging"
source_lesson: "redis-messaging-consumer-groups-intro"
---

# Introduction to Consumer Groups

## Introduction

Consumer groups allow multiple consumers to cooperate in processing a stream. Each message is delivered to exactly one consumer in the group, enabling parallel processing without duplicates.

## Key Concepts

- **Consumer group**: a named group of consumers that share the processing of a stream; each message goes to exactly one member
- **XGROUP CREATE**: creates a consumer group with a starting ID (`0` for beginning, `$` for new messages only)
- **XREADGROUP**: reads messages as a named consumer within a group; `>` means undelivered messages only
- **Pending Entries List (PEL)**: tracks messages that have been delivered but not yet acknowledged

## Real World Context

- **Order processing**: 5 worker pods share a consumer group — each order is processed once
- **Email sending**: 3 email workers pull from the same group — no duplicate emails
- **Image resizing**: parallel workers handle different uploaded images simultaneously

## Deep Dive

With plain `XREAD`, every consumer reading from a stream receives all messages — like a broadcast. Consumer groups change this to a work-queue model: each message goes to exactly one worker.

```
Stream: [msg1] [msg2] [msg3] [msg4] [msg5]

With consumer group:
  Worker 1 gets: msg1, msg3, msg5
  Worker 2 gets: msg2, msg4
```

```bash
# Create group starting from the beginning
XGROUP CREATE orders processors 0

# Create group for new messages only, create stream if absent
XGROUP CREATE orders processors $ MKSTREAM

# Read up to 10 undelivered messages for consumer worker-1
XREADGROUP GROUP processors worker-1 COUNT 10 STREAMS orders >
```

## Common Pitfalls

- **Using `0` vs `$` at creation**: `0` replays all existing messages; `$` only delivers future messages — pick based on whether you need history
- **Forgetting XACK**: messages stay in the PEL until acknowledged; accumulating pending messages degrades performance
- **Consumer name collisions**: if two processes use the same consumer name, they compete for the same pending entries

## Best Practices

- Use `MKSTREAM` in development to avoid errors when the stream does not exist yet
- Use unique, stable consumer names (e.g., hostname or pod name) to track pending messages per worker
- Always call XACK after successful processing — never after a failed attempt

## Summary

Consumer groups transform a stream into a distributed work queue. Use `XGROUP CREATE` to create the group, `XREADGROUP` with `>` to claim new messages, and `XACK` to confirm processing.

## Code Examples

**Consumer group setup**

```bash
# Create stream and consumer group
XADD orders '*' status pending customerId 1
XGROUP CREATE orders processors $ MKSTREAM

# Worker 1 reads its share of messages
XREADGROUP GROUP processors worker-1 COUNT 5 STREAMS orders >

# Worker 2 reads its share
XREADGROUP GROUP processors worker-2 COUNT 5 STREAMS orders >
```


## Resources

- [Redis Streams Tutorial](https://redis.io/docs/latest/develop/data-types/streams-tutorial/) — Comprehensive Redis Streams tutorial covering consumer groups

---

> 📘 *This lesson is part of the [Real-Time Messaging and Event Streaming](https://stanza.dev/courses/redis-messaging) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*