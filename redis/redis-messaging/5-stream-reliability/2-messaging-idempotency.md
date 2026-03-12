---
source_course: "redis-messaging"
source_lesson: "redis-messaging-idempotency"
---

# Idempotency in Stream Processing

## Introduction

Because Redis Streams provide at-least-once delivery, the same message may be processed more than once (after a crash and reclaim). Idempotent processing ensures this causes no harm.

## Key Concepts

- **Idempotent operation**: produces the same result whether performed once or many times (e.g., `SET balance 100`)
- **Non-idempotent operation**: produces different results on repeated calls (e.g., `INCRBY balance 100`)
- **Idempotency key**: a unique identifier used to detect duplicate processing; the stream entry ID is a natural key
- **SETNX dedup**: use `SET key 1 NX EX ttl` to atomically mark a message as being processed

## Real World Context

Payment systems always implement idempotency: if a network timeout causes a message to be redelivered, you do not want to charge a customer twice. The stream entry ID serves as the payment idempotency key.

## Deep Dive

```bash
# Entry ID is globally unique: 1691234567890-0
# Use it as a dedup key in Redis
SETNX processed:1691234567890-0 1 EX 86400
# If result is 1 -> not seen before, process it
# If result is 0 -> already processed, skip
```

Database upsert pattern:

```sql
INSERT INTO orders (id, status) VALUES ($orderId, $status)
ON CONFLICT (id) DO NOTHING;
```

| Operation | Idempotent Version |
|---|---|
| Send email | Check `email_sent:{msgId}` flag before sending |
| Charge payment | Use idempotency key in payment API |
| Update inventory | SET to absolute value, not relative |

## Common Pitfalls

- **Using relative updates (INCRBY, INCR)**: if the message is replayed, the count is incremented twice
- **Forgetting to set TTL on dedup keys**: keys accumulate forever and waste memory
- **Race condition in check-then-act**: checking a dedup flag and then acting is not atomic; use SETNX or a Lua script

## Best Practices

- Use the stream entry ID as the idempotency key — it is unique, time-ordered, and always available
- Prefer database-level unique constraints over Redis dedup keys for durability
- Design all message handlers to be idempotent by default, not as an afterthought

## Summary

Idempotency is a contract your processing code must uphold. Use the stream entry ID as a natural idempotency key. Common patterns: Redis SETNX dedup, database unique constraints, or absolute-value updates instead of relative increments.

## Code Examples

**Deduplication with SETNX**

```bash
# Processing entry 1691234567890-0

# Attempt to mark as processing (atomic)
SETNX processing:1691234567890-0 1
# => 1: first time, proceed
# => 0: already being processed or done, skip

# Set expiry so the key eventually cleans up
EXPIRE processing:1691234567890-0 3600

# After successful processing, acknowledge
XACK orders processors 1691234567890-0

# Optional: move to completed set for audit
SADD completed:orders 1691234567890-0
```


## Resources

- [Redis Streams](https://redis.io/docs/latest/develop/data-types/streams/) — Official Redis Streams documentation

---

> 📘 *This lesson is part of the [Real-Time Messaging and Event Streaming](https://stanza.dev/courses/redis-messaging) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*