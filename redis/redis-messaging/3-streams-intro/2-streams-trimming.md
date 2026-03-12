---
source_course: "redis-messaging"
source_lesson: "redis-messaging-streams-trimming"
---

# Stream Trimming and Memory Management

## Introduction

Streams grow indefinitely by default. For production systems, you need to trim them to prevent unbounded memory growth.

## Key Concepts

- **XTRIM MAXLEN**: keeps only the last N entries; `~` flag enables approximate trimming
- **XTRIM MINID**: removes all entries with an ID less than the given timestamp-based ID (time-based retention)
- **Inline trimming**: passing `MAXLEN` or `MINID` directly to XADD combines append and trim atomically
- **Approximate trimming (`~`)**: allows Redis to keep slightly more entries to avoid expensive radix-tree rebalancing

## Real World Context

- **IoT sensor stream**: keep last 10,000 readings per device
- **Activity feed**: keep 30 days of events using MINID with a rolling timestamp
- **High-frequency trading**: keep last 1 minute of ticks, trim on every XADD

## Deep Dive

```bash
# Keep only the last 1000 entries (exact)
XTRIM mystream MAXLEN 1000

# Approximate trim (faster, may keep slightly more)
XTRIM mystream MAXLEN ~ 1000

# Trim by minimum ID (remove entries older than a timestamp)
XTRIM mystream MINID 1691000000000

# Add entry AND keep stream at ~1000 entries (atomic)
XADD mystream MAXLEN ~ 1000 '*' field value
```

| Strategy | Command | When to Use |
|---|---|---|
| Keep N entries | `MAXLEN 1000` | Fixed-size sliding window |
| Keep by age | `MINID timestamp` | Time-based retention (e.g., 7 days) |

Each stream entry consumes memory proportional to the number of fields. Use `OBJECT ENCODING mystream` to inspect the internal encoding (listpack for small streams, stream for large ones).

## Common Pitfalls

- **Not trimming at all**: streams that grow without bound will eventually exhaust Redis memory
- **Using exact MAXLEN on high-throughput streams**: every exact trim rebalances the radix tree; use `~` for performance
- **Trimming without considering consumer group lag**: trimming entries that a consumer group hasn't processed yet causes those entries to be lost

## Best Practices

- Always use inline MAXLEN trimming (`XADD stream MAXLEN ~ N * ...`) rather than a separate XTRIM call
- Use MINID for time-based retention to align with your data lifecycle policy
- Monitor stream length with `XLEN` and set alerts if it grows beyond expected bounds

## Summary

Always trim production streams. Use `MAXLEN ~` for count-based retention and `MINID` for time-based retention. Combining trimming with XADD is the most efficient pattern — you maintain the stream size with no additional round trips.

## Code Examples

**Stream trimming patterns**

```bash
# Exact trim to 1000 entries
XTRIM events MAXLEN 1000

# Approximate trim (much faster on large streams)
XTRIM events MAXLEN ~ 1000

# Trim by age: remove entries before 7 days ago
# (calculate UNIX milliseconds for 7 days ago)
XTRIM events MINID ~ 1690629567000

# Trim inline with XADD
XADD events MAXLEN ~ 5000 '*' type click userId 42
```


## Resources

- [Redis Streams](https://redis.io/docs/latest/develop/data-types/streams/) — Official Redis Streams data type documentation

---

> 📘 *This lesson is part of the [Real-Time Messaging and Event Streaming](https://stanza.dev/courses/redis-messaging) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*