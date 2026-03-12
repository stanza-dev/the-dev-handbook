---
source_course: "redis-messaging"
source_lesson: "redis-messaging-consumer-group-management"
---

# Managing Consumer Groups

## Introduction

Beyond creating groups and reading messages, Redis provides commands to manage the lifecycle of consumer groups and individual consumers.

## Key Concepts

- **XGROUP SETID**: resets the last-delivered-ID cursor for a group, enabling replay or fast-forward
- **XGROUP DELCONSUMER**: removes a consumer from the group; its pending messages remain in the PEL under the group
- **XGROUP DESTROY**: deletes the entire consumer group including its PEL (does not delete the stream)
- **XINFO GROUPS / CONSUMERS**: read-only inspection commands for group and consumer metadata

## Real World Context

- **Blue-green deployment**: use XGROUP SETID to replay messages to a new version of your service before switching traffic
- **Scaling down**: XGROUP DELCONSUMER safely removes a pod's consumer entry during scale-in
- **Monitoring**: XINFO GROUPS shows lag (pending messages) per group for dashboards

## Deep Dive

```bash
# Reset cursor to replay from the beginning
XGROUP SETID mystream mygroup 0

# Remove a consumer (its pending messages stay in the group PEL)
XGROUP DELCONSUMER mystream mygroup consumer1
# => (integer) 3  <- number of pending messages formerly held by this consumer

# Delete the entire group (does not delete the stream)
XGROUP DESTROY mystream mygroup
# => (integer) 1

# List all groups on a stream
XINFO GROUPS mystream

# List consumers in a group
XINFO CONSUMERS mystream mygroup
```

## Common Pitfalls

- **Using XGROUP DESTROY carelessly**: it deletes the PEL, meaning all pending messages are lost — unprocessed work disappears
- **Not re-claiming messages after DELCONSUMER**: pending messages left after removing a consumer must be claimed via XCLAIM or they will accumulate indefinitely
- **Forgetting MKSTREAM**: `XGROUP CREATE` on a non-existent stream fails unless you pass `MKSTREAM`

## Best Practices

- Use `XINFO GROUPS` in dashboards to monitor pending message counts per group (a good indicator of consumer lag)
- Before destroying a group, drain its PEL by claiming and processing all pending messages
- Use XGROUP SETID 0 before running replay tests to ensure full coverage

## Summary

XGROUP SETID lets you rewind or fast-forward a group's read position. XGROUP DELCONSUMER safely removes a consumer and releases its pending messages. XINFO GROUPS/CONSUMERS are the primary observability tools for consumer group health.

## Code Examples

**Consumer group management**

```bash
# Create group
XGROUP CREATE orders processors $ MKSTREAM

# Inspect groups on the stream
XINFO GROUPS orders
# => pending, last-delivered-id, consumers count, etc.

# List consumers
XINFO CONSUMERS orders processors

# Reset cursor to replay from the beginning
XGROUP SETID orders processors 0

# Remove a crashed consumer
XGROUP DELCONSUMER orders processors worker-dead

# Delete the group when no longer needed
XGROUP DESTROY orders processors
```


## Resources

- [Redis Streams Tutorial](https://redis.io/docs/latest/develop/data-types/streams-tutorial/) — Comprehensive Redis Streams tutorial covering consumer groups

---

> 📘 *This lesson is part of the [Real-Time Messaging and Event Streaming](https://stanza.dev/courses/redis-messaging) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*