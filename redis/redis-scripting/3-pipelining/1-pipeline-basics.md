---
source_course: "redis-scripting"
source_lesson: "redis-scripting-pipeline-basics"
---

# How Pipelining Works

## Introduction

Pipelining lets a Redis client send multiple commands to the server without waiting for the response to each one. Instead of paying a network round-trip for every command, the client buffers all commands and sends them in a single batch, then reads all responses at once. This can reduce latency by an order of magnitude in high-throughput scenarios.

## Key Concepts

- **Round-trip time (RTT)** — the network latency for one request/response cycle; the main bottleneck when sending many commands
- **Pipelining** — buffering multiple commands client-side and sending them in one TCP write
- **Throughput vs. latency** — pipelining maximizes throughput; individual command latency is unchanged
- **Not atomic** — the server may execute other clients' commands between pipelined commands

## Real World Context

An analytics job needs to increment 10,000 counters at startup. Without pipelining, each INCR requires an RTT (say 1 ms each) — 10 seconds total. With pipelining, all 10,000 INCRs are sent in a few TCP packets and responses arrive in bulk — the whole job completes in milliseconds.

## Deep Dive

At the protocol level, pipelining is simply sending multiple RESP commands without waiting for responses between them:

```bash
# Without pipelining: 3 round trips
SET key1 "a"  # wait...
SET key2 "b"  # wait...
SET key3 "c"  # wait...

# With pipelining: 1 round trip
# Client sends all three commands at once,
# reads all three responses at once
```

In Python with redis-py:

```python
import redis

r = redis.Redis()

# Open a pipeline (transaction=False means no MULTI/EXEC)
with r.pipeline(transaction=False) as pipe:
    for i in range(10000):
        pipe.incr(f"counter:{i}")
    results = pipe.execute()  # Sends all 10,000 commands, reads all responses

# results is a list of 10,000 integers
```

The number of round trips is 1 regardless of how many commands are pipelined. The server still processes each command sequentially and other clients' commands can interleave between them.

## Common Pitfalls

1. **Assuming pipelining is atomic.** It is not. Other clients can write between your pipelined commands. Use MULTI/EXEC inside a pipeline for atomicity.
2. **Pipelining too many commands at once.** Very large pipelines consume significant server memory to buffer responses. Batch in chunks of 1,000-10,000 commands.
3. **Ignoring per-command errors in the response list.** If one command fails, redis-py raises an exception when iterating results. Check each result.

## Best Practices

1. Use pipelines for bulk operations (seeding, migrations, analytics) where you have many independent commands.
2. Batch large pipelines into chunks of 1,000-10,000 to avoid memory pressure.
3. Combine pipelining with MULTI/EXEC when you need both performance and atomicity.

## Summary

- Pipelining sends multiple commands in one round trip instead of one per command
- RTT reduction is the primary benefit — server-side execution time is unchanged
- Pipelining is NOT atomic; interleaving from other clients is possible
- Use transaction=False in redis-py for pure pipelining without MULTI/EXEC
- Batch large pipelines into chunks to control memory usage

## Code Examples

**Bulk increment using redis-py pipeline**

```python
import redis

r = redis.Redis()

with r.pipeline(transaction=False) as pipe:
    for i in range(10000):
        pipe.incr(f"counter:{i}")
    results = pipe.execute()

print(f"Completed {len(results)} increments in one round trip")
```


## Resources

- [Redis Pipelining](https://redis.io/docs/latest/develop/use/pipelining/) — Guide to Redis pipelining

---

> 📘 *This lesson is part of the [Scripting, Transactions, and Programmability](https://stanza.dev/courses/redis-scripting) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*