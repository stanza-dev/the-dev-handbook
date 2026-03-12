---
source_course: "redis-messaging"
source_lesson: "redis-messaging-dead-letter"
---

# Handling Poison Messages with Dead-Letter Patterns

## Introduction

A poison message is one that consistently fails processing. Without a dead-letter strategy, it blocks the consumer group indefinitely, holding up other messages.

## Key Concepts

- **Poison message**: a stream entry that repeatedly fails processing; identified by a high delivery count in XPENDING
- **Dead-Letter Queue (DLQ)**: a separate stream (`orders:dlq`) where poison messages are moved for inspection and replay
- **Max retry threshold**: the delivery count at which a message is considered unprocessable and rerouted to the DLQ
- **Error-handler consumer**: a specialized consumer that receives claimed poison messages, logs them, and acknowledges them

## Real World Context

- AWS SQS, Google Pub/Sub, RabbitMQ all have native DLQ support — implement the same pattern with Redis Streams manually
- Dead-letter streams enable post-mortem analysis: inspect what caused the failure, fix the bug, then replay from the DLQ

## Deep Dive

```bash
# 1. Detect poison message via high delivery count
XPENDING orders processors - + 10
# ID                  Consumer  Idle(ms)  Deliveries
# 1691234567890-0     worker-1  300000    12  <- poison message!

# 2. Read the message details
XRANGE orders 1691234567890-0 1691234567890-0

# 3. Append to dead-letter stream with error context
XADD orders:dlq '*' original-id 1691234567890-0 error "NullPointerException" attempts 12

# 4. Acknowledge and remove from main stream
XACK orders processors 1691234567890-0
```

Consumer loop with retry threshold:

```
read message → if delivery_count >= MAX_RETRIES:
  move to DLQ stream
  XACK original
else:
  process normally
  XACK on success
```

## Common Pitfalls

- **Not setting a retry threshold**: without a limit, a poison message is reclaimed and retried forever
- **Moving to DLQ without XACK**: the message stays in the PEL even after DLQ insertion, causing duplicate entries
- **Losing error context**: always append failure reason and delivery count to the DLQ entry for post-mortem analysis

## Best Practices

- Always include original message ID, error description, and attempt count when writing to the DLQ stream
- Configure alerts when the DLQ stream grows beyond a threshold
- After fixing the bug, replay the DLQ: `XRANGE orders:dlq - +` and reprocess each entry

## Summary

Detect poison messages via XPENDING delivery counts. Move them to a dedicated dead-letter stream with error context, then XACK them from the main stream. This prevents one bad message from blocking all subsequent processing.

## Code Examples

**Dead-letter stream pattern**

```bash
# Check delivery count
XPENDING orders processors - + 1
# => 1691234567890-0  worker-1  300000ms  15 deliveries

# Move to dead-letter queue
XADD orders:dlq '*' \
  original-id 1691234567890-0 \
  error "Failed after 15 attempts" \
  first-seen 1691234567890

# Remove from active processing
XACK orders processors 1691234567890-0

# Later: replay from DLQ after fixing the bug
XRANGE orders:dlq - + COUNT 100
```


## Resources

- [Redis Streams](https://redis.io/docs/latest/develop/data-types/streams/) — Official Redis Streams documentation

---

> 📘 *This lesson is part of the [Real-Time Messaging and Event Streaming](https://stanza.dev/courses/redis-messaging) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*