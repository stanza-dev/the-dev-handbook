---
source_course: "redis-messaging"
source_lesson: "redis-messaging-event-sourcing"
---

# Event Sourcing with Redis Streams

## Introduction

Event sourcing is an architectural pattern where the state of an entity is derived entirely from a sequence of immutable events — rather than stored as a mutable record. Redis Streams map naturally to this pattern.

## Key Concepts

- **Event store**: the append-only log (Redis Stream) that holds all state-change events
- **Immutability**: stream entries are not modified after creation; current state is always derived by replaying
- **Replay**: processing all events in order via XRANGE to reconstruct current state
- **Snapshot**: a cached state at a known stream ID to speed up replay for long streams

## Real World Context

Banking: account balance is derived from a stream of transactions (deposit, withdrawal). The stream IS the source of truth — not a balance column.

## Deep Dive

Instead of: `UPDATE orders SET status = 'shipped' WHERE id = 1`

You do: `XADD order:1 '*' event OrderShipped shippedAt 2024-01-15T10:00:00Z`

The current state is reconstructed by replaying all events in order:

```
OrderPlaced → OrderPaid → OrderShipped → (current state: shipped)
```

```bash
# Append events as they happen
XADD order:1 '*' event OrderPlaced customerId 42 amount 99.95
XADD order:1 '*' event OrderPaid paymentId pay-7
XADD order:1 '*' event OrderShipped trackingId TRK-123

# Reconstruct state: replay all events
XRANGE order:1 - +
# Process each event in sequence to build current state
```

Why Streams fit event sourcing:
- **Immutable**: entries cannot be modified (only deleted, which is discouraged in event sourcing)
- **Ordered**: IDs guarantee chronological order
- **Persistent**: history is never lost
- **Replayable**: XRANGE lets you rebuild state from any point

## Common Pitfalls

- **Storing mutable state alongside events**: if you also keep a current-state field, it can diverge from the event log
- **Using XDEL**: deleting events corrupts history; use soft-delete events (e.g., `OrderCancelled`) instead
- **Not handling schema evolution**: event shapes change over time; consumers must handle both old and new formats

## Best Practices

- Use past-tense event names (OrderPlaced, not PlaceOrder) to convey immutability
- Take periodic snapshots to bound replay time for long-lived entities
- Version your event schema from day one to make future migrations easier

## Summary

Event sourcing treats every state change as an immutable event appended to a log. Redis Streams provide the persistence, ordering, and replay capabilities needed. Current state is always derived, never stored — reducing write conflicts and enabling full history.

## Code Examples

**Event sourcing with Redis Streams**

```bash
# Append domain events
XADD account:1001 '*' event MoneyDeposited amount 500 balance 500
XADD account:1001 '*' event MoneyWithdrawn amount 100 balance 400
XADD account:1001 '*' event MoneyDeposited amount 250 balance 650

# Replay to reconstruct current balance
XRANGE account:1001 - +
# Apply each event to derive: balance = 650

# Reconstruct state at a specific point in time
XRANGE account:1001 - 1691234567890 +
# Balance before that timestamp
```


## Resources

- [Redis Streams](https://redis.io/docs/latest/develop/data-types/streams/) — Official Redis Streams documentation with event sourcing patterns

---

> 📘 *This lesson is part of the [Real-Time Messaging and Event Streaming](https://stanza.dev/courses/redis-messaging) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*