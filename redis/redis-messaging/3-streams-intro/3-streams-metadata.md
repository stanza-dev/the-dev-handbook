---
source_course: "redis-messaging"
source_lesson: "redis-messaging-streams-metadata"
---

# Stream Entry IDs and Metadata Operations

## Introduction

Beyond reading and writing, Redis Streams provide commands to inspect, navigate, and manage stream contents.

## Key Concepts

- **XLEN**: returns the total number of entries in a stream
- **XRANGE**: reads entries in ascending order between two IDs; `-` is the minimum, `+` is the maximum
- **XREVRANGE**: reads entries in descending order (newest first); start and end arguments are reversed
- **XDEL**: marks an entry as deleted (does not immediately reclaim memory)
- **Partial IDs**: `1691234567890` is shorthand for `1691234567890-0`

## Real World Context

- **Admin dashboard**: XLEN shows queue depth at a glance
- **Replay audits**: XRANGE from a specific timestamp to reconstruct what happened at a point in time
- **Pagination**: cursor-based pagination using the last received ID as the next XRANGE start

## Deep Dive

Stream IDs follow the pattern `milliseconds-sequenceNumber`:

```
1691234567890-0   <- first entry at millisecond 1691234567890
1691234567890-1   <- second entry at the SAME millisecond
1691234567891-0   <- next millisecond
```

```bash
# Count entries
XLEN mystream
# => (integer) 4201

# All entries
XRANGE mystream - +

# Paginate: first 10 entries
XRANGE mystream - + COUNT 10

# Next page: start after the last ID received (exclusive)
XRANGE mystream '(1691234567890-1' + COUNT 10

# Read entries newest-first
XREVRANGE mystream + - COUNT 5
```

XDEL does not reclaim memory immediately — it marks the entry as deleted. Use XTRIM to actually reclaim space. In event sourcing contexts, XDEL is generally avoided.

## Common Pitfalls

- **Including the boundary ID on pagination**: use the `(ID` exclusive syntax to avoid re-processing the last seen entry
- **Forgetting reversed arguments in XREVRANGE**: `XREVRANGE stream + -` is correct; `XREVRANGE stream - +` reads nothing
- **Confusing XDEL and XTRIM**: XDEL removes a specific entry; XTRIM removes a range of entries and reclaims memory

## Best Practices

- Use `XRANGE` with `COUNT` and cursor-based pagination for large stream reads
- Prefer `XREVRANGE` for recent-activity UIs (news feeds, activity logs)
- Use `(ID` exclusive syntax for correct cursor pagination without off-by-one errors

## Summary

XLEN gives a fast count. XRANGE reads forward by ID range and is the foundation for cursor-based pagination. XREVRANGE reads backward for most-recent-first displays. Use partial IDs (just the millisecond part) as shorthand when sequence number precision is not needed.

## Code Examples

**Metadata and range operations**

```bash
# Count entries
XLEN orders
# => (integer) 4201

# Read all entries
XRANGE orders - +

# Paginate: 10 per page
XRANGE orders - + COUNT 10
# => last entry ID: 1691234567890-3

# Next page (exclusive: start AFTER last seen ID)
XRANGE orders '(1691234567890-3' + COUNT 10

# Last 5 entries (reverse)
XREVRANGE orders + - COUNT 5

# Stream entry count
XLEN orders
```


## Resources

- [Redis Streams](https://redis.io/docs/latest/develop/data-types/streams/) — Official Redis Streams data type documentation

---

> 📘 *This lesson is part of the [Real-Time Messaging and Event Streaming](https://stanza.dev/courses/redis-messaging) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*