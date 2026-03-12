---
source_course: "redis-scripting"
source_lesson: "redis-scripting-pipelining"
---

# Understanding Pipelining

## Introduction

Pipelining is the technique of sending multiple Redis commands without waiting for individual responses, dramatically reducing latency for bulk operations. It works at the network protocol level and requires no special server configuration.

## Key Concepts

- **Network latency (RTT)** — the delay for each request/response cycle; the main bottleneck in command-heavy workloads
- **Pipelining** — client buffers commands and sends them together; reads all responses in one batch
- **Throughput improvement** — pipelining can increase throughput by 5-10x in high-latency networks
- **No atomicity guarantee** — pipelining alone does not prevent other clients from interleaving

## Real World Context

A social network builds a user's feed by fetching 50 post records. Without pipelining: 50 RTTs × 1ms each = 50ms minimum. With pipelining: 1 RTT = ~1ms. The difference is dramatic for users on the other side of the world.

## Deep Dive

The network overhead comparison:

```bash
# Sequential (N round trips for N commands):
GET post:1    # RTT 1
GET post:2    # RTT 2
...           # RTT N

# Pipelined (1 round trip for N commands):
# Client sends: GET post:1\r\nGET post:2\r\n...GET post:N\r\n
# Server responds with all N results in one TCP stream
```

Python example fetching multiple posts:

```python
import redis

r = redis.Redis()

post_ids = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

with r.pipeline(transaction=False) as pipe:
    for post_id in post_ids:
        pipe.hgetall(f"post:{post_id}")
    posts = pipe.execute()  # List of 10 dicts, fetched in one round trip
```

To combine pipelining with atomicity, wrap the commands in MULTI/EXEC inside the pipeline:

```python
with r.pipeline(transaction=True) as pipe:  # transaction=True adds MULTI/EXEC
    pipe.incr("visits")
    pipe.incr("unique_visitors")
    results = pipe.execute()  # Atomic AND pipelined
```

## Common Pitfalls

1. **Using pipelining for dependent commands.** If command B depends on command A's result, you cannot pipeline them — you need A's result before constructing B.
2. **Confusing pipeline performance with Lua performance.** A Lua script with 10 Redis calls uses fewer round trips than 10 individual commands, but is similar to one pipelined batch. Lua adds server-side logic; pipelining reduces RTT.
3. **Not handling partial failures.** In a large pipeline, some commands may fail. Always inspect the results list.

## Best Practices

1. Pipeline reads (GET, HGETALL, LRANGE) that are independent of each other for maximum throughput.
2. Use transaction=True (MULTI/EXEC) when the pipelined commands must execute atomically.
3. Profile before optimizing — pipelining helps most when RTT is high (remote Redis, cloud deployments).

## Summary

- Pipelining batches multiple commands into one network round trip
- Server-side processing time is unchanged; only RTT is reduced
- NOT atomic — other clients can interleave between pipelined commands
- Combine pipeline with MULTI/EXEC (transaction=True) for atomic batches
- Most useful when Redis is remote or RTT is significant

## Code Examples

**Fetching multiple keys with pipelining**

```python
import redis

r = redis.Redis()

# Fetch 10 posts in one round trip
post_ids = list(range(1, 11))
with r.pipeline(transaction=False) as pipe:
    for post_id in post_ids:
        pipe.hgetall(f"post:{post_id}")
    posts = pipe.execute()

print(f"Fetched {len(posts)} posts in 1 round trip")
```


## Resources

- [Redis Pipelining](https://redis.io/docs/latest/develop/use/pipelining/) — Guide to Redis pipelining

---

> 📘 *This lesson is part of the [Scripting, Transactions, and Programmability](https://stanza.dev/courses/redis-scripting) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*