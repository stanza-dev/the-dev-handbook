---
source_course: "redis-messaging"
source_lesson: "redis-messaging-consumer-groups-ack"
---

# Acknowledging Messages

## Introduction

When a consumer reads a message via XREADGROUP, Redis adds it to the Pending Entries List (PEL) — a per-group tracking structure. The message stays there until explicitly acknowledged.

## Key Concepts

- **PEL (Pending Entries List)**: per-group structure tracking all delivered-but-unacknowledged messages with consumer name, idle time, and delivery count
- **XACK**: removes one or more entries from the PEL, signaling successful processing
- **At-least-once delivery**: because messages stay in the PEL until acknowledged, a crashed consumer's messages can be reclaimed and reprocessed
- **Idempotency**: processing the same message twice must produce the same result

## Real World Context

Payment processing: after charging a customer, call XACK. If the worker crashes before XACK, another worker reclaims the message. Since the payment API is called again, idempotency keys prevent a double charge.

## Deep Dive

The PEL tracks:
- Which messages have been delivered
- Which consumer has each message
- When each message was last delivered
- How many times each message has been delivered

```bash
# After successfully processing a message:
XACK orders processors 1691234567890-0
# => (integer) 1  <- number of entries acknowledged

# Acknowledge multiple at once
XACK orders processors 1691234567890-0 1691234567891-0 1691234567892-0

# Check pending messages
XPENDING orders processors - + 10
```

Once acknowledged, the entry is removed from the PEL (but remains in the stream itself until trimmed).

## Common Pitfalls

- **Calling XACK before processing is complete**: if the process crashes after XACK but before finishing, the message is lost
- **Not calling XACK on error**: messages accumulate in the PEL indefinitely; build a dead-letter pattern instead
- **Batch-acknowledging without processing**: acknowledging messages you haven't processed defeats at-least-once delivery

## Best Practices

- Call XACK only after all side effects of processing are complete and durable
- On processing failure, do NOT acknowledge; let the PEL accumulate idle time so XCLAIM can recover the message
- Monitor `XPENDING` regularly to detect consumers that are not acknowledging messages

## Summary

XACK removes a message from the PEL, signaling successful processing. Unacknowledged messages remain tracked and can be reclaimed. The combination of PEL + XACK gives Redis Streams at-least-once delivery semantics.

## Code Examples

**Acknowledge messages**

```bash
# Read a message
XREADGROUP GROUP processors worker-1 COUNT 1 STREAMS orders >
# => 1) 1) "orders"
#    2) 1) 1) "1691234567890-0"
#          2) 1) "status" 2) "pending"

# Process it... then acknowledge
XACK orders processors 1691234567890-0
# => (integer) 1

# Check what's still pending
XPENDING orders processors - + 10
```


## Resources

- [Redis Streams Tutorial](https://redis.io/docs/latest/develop/data-types/streams-tutorial/) — Comprehensive Redis Streams tutorial covering consumer groups

---

> 📘 *This lesson is part of the [Real-Time Messaging and Event Streaming](https://stanza.dev/courses/redis-messaging) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*