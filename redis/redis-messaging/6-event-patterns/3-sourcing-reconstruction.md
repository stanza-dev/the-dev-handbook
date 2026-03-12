---
source_course: "redis-messaging"
source_lesson: "redis-messaging-event-sourcing-reconstruction"
---

# Reconstructing State from Event Streams

## Introduction

State reconstruction — replaying stored events to derive current state — is the core operation in event sourcing. Redis Streams provide the primitives needed to do this efficiently.

## Key Concepts

- **Full replay**: read all events from `0` to `+` and apply them sequentially to derive current state
- **Cursor-based pagination**: use `(lastId + COUNT N` to read large streams in pages without loading everything into memory
- **Exclusive ID syntax**: `(ID` prefix means "start after this ID" — prevents re-processing the boundary entry
- **Snapshot + partial replay**: load a checkpoint state and replay only events after the snapshot ID to reduce latency

## Real World Context

- Banking systems snapshot balances nightly and replay only intraday transactions on startup
- E-commerce reconstructs order state from the last snapshot + new events on each API call
- Analytics systems replay events into materialized views during backfills

## Deep Dive

```bash
# Page 1: first 500 events
XRANGE account:1001 - + COUNT 500
# Last received ID: 1691234567890-3

# Page 2: next 500 (exclusive start)
XRANGE account:1001 '(1691234567890-3' + COUNT 500
# Repeat until empty result
```

Starting from a snapshot:

```bash
# Load snapshot (stored in a Redis Hash)
HGETALL snapshot:account:1001
# => state at stream ID 1691200000000-0, balance: 12500

# Replay only events AFTER the snapshot
XRANGE account:1001 '(1691200000000-0' + COUNT 500
# Apply delta events to snapshot state to get current balance
```

| Approach | Pro | Con |
|---|---|---|
| Full replay | Simple | Slow for long streams |
| Snapshot + partial replay | Fast | Extra complexity, snapshot freshness |
| Periodic snapshots | Balanced | Must manage snapshot lifecycle |

## Common Pitfalls

- **Inclusive boundary on pagination**: using `XRANGE stream lastId +` re-processes the last entry; use `(lastId` to be exclusive
- **Taking snapshots too infrequently**: if the stream is trimmed and the snapshot is older than the trim point, data is lost
- **Not handling empty pages**: the replay loop must stop when XRANGE returns an empty array, not on a fixed count

## Best Practices

- Always use the exclusive `(ID` syntax for cursor-based pagination
- Take snapshots before trimming: snapshot at the current stream tip, then trim up to that ID
- Test replay performance: if reconstructing state takes >100ms, add a snapshot

## Summary

Use XRANGE with cursor pagination for large stream replay. Start from a snapshot ID to skip replaying historical events. The `(ID` exclusive syntax is key for correct cursor-based pagination without re-processing the boundary entry.

## Code Examples

**Cursor-based state reconstruction**

```bash
# Full replay with cursor pagination
XRANGE account:1001 - + COUNT 500
# Process events, note last ID: 1691234567890-3

XRANGE account:1001 '(1691234567890-3' + COUNT 500
# Process next batch...
# Repeat until empty

# Start from snapshot
# Load snapshot state from:
HGETALL snapshot:account:1001
# { stream_id: 1691200000000-0, balance: 12500, ... }

# Replay only events after snapshot
XRANGE account:1001 '(1691200000000-0' + COUNT 500
# Apply delta events to snapshot state
```


## Resources

- [Redis Streams](https://redis.io/docs/latest/develop/data-types/streams/) — Official Redis Streams documentation with event sourcing patterns

---

> 📘 *This lesson is part of the [Real-Time Messaging and Event Streaming](https://stanza.dev/courses/redis-messaging) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*