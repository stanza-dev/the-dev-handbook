---
source_course: "redis-messaging"
source_lesson: "redis-messaging-failure-handling"
---

# Handling Consumer Failures

## Introduction

In distributed stream processing, consumers will fail. Redis Streams provides the Pending Entries List (PEL) and XCLAIM to detect and recover from these failures.

## Key Concepts

- **XPENDING**: inspects the PEL to find delivered-but-unacknowledged messages per consumer; the `IDLE` option filters by minimum idle time
- **XCLAIM**: transfers ownership of a specific pending message from one consumer to another, resetting its idle timer
- **XAUTOCLAIM**: atomically scans the PEL for idle messages and claims them in one operation (Redis 6.2+)
- **Delivery count**: increments each time a message is delivered or claimed; high counts signal a poison message

## Real World Context

- **Pod crashes in Kubernetes**: when a pod dies, a health-check loop calls XAUTOCLAIM to recover its messages
- **Long-running tasks**: if processing takes >30s, another worker may claim the message — idempotency prevents double-processing

## Deep Dive

```bash
# Summary: how many messages each consumer has pending
XPENDING orders processors

# Detail: messages pending for more than 60 seconds
XPENDING orders processors IDLE 60000 - + 100
# Returns: message ID, consumer name, idle time (ms), delivery count

# Manually claim a specific stuck message
XCLAIM orders processors worker-2 60000 1691234567890-0

# XAUTOCLAIM: automatically find and claim old messages
XAUTOCLAIM orders processors worker-2 60000 0 COUNT 10
```

A high delivery count indicates a poison message:

```
ID                  Consumer  Idle(ms)  Delivery-count
1691234567890-0     worker-1  120000    5   <- reclaim or dead-letter this!
```

## Common Pitfalls

- **Claiming too aggressively**: setting the min-idle-time too low can claim messages still being processed by slow consumers
- **Not checking delivery count before claiming**: reclaiming a poison message just moves the problem; check the count and dead-letter if it exceeds a threshold
- **Running reclaim logic on every worker**: only one designated reclaim worker (or a scheduled job) should run XAUTOCLAIM to avoid races

## Best Practices

- Run a separate watchdog process that calls XAUTOCLAIM periodically (e.g., every 30 seconds)
- Set a max retry threshold; move messages exceeding it to a dead-letter stream
- Log the delivery count when claiming, so you have visibility into retry history

## Summary

Use XPENDING to detect stuck messages, XCLAIM/XAUTOCLAIM to reassign them to healthy consumers. Always monitor delivery counts to identify poison messages that repeatedly fail.

## Code Examples

**Failure detection and recovery**

```bash
# Find messages pending >30 seconds
XPENDING orders processors IDLE 30000 - + 50

# Manually claim a specific stuck message
XCLAIM orders processors worker-2 30000 1691234567890-0

# Auto-claim: find and reassign in one operation
XAUTOCLAIM orders processors worker-2 30000 0 COUNT 10
# => next cursor + list of claimed entries

# Acknowledge after processing
XACK orders processors 1691234567890-0
```


## Resources

- [Redis Streams](https://redis.io/docs/latest/develop/data-types/streams/) — Official Redis Streams documentation

---

> 📘 *This lesson is part of the [Real-Time Messaging and Event Streaming](https://stanza.dev/courses/redis-messaging) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*