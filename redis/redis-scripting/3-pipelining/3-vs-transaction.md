---
source_course: "redis-scripting"
source_lesson: "redis-scripting-pipeline-vs-transaction"
---

# Pipelining vs. Transactions

## Introduction

Pipelining and MULTI/EXEC transactions are often confused, but they solve different problems. Pipelining reduces network overhead; MULTI/EXEC provides atomicity. Understanding their differences — and how to combine them — is essential for building correct, high-performance Redis applications.

## Key Concepts

- **Pipelining** — a client-side batching optimization; reduces RTT; NOT atomic
- **MULTI/EXEC transaction** — guarantees no other client commands interleave; slower per command than pipelining alone
- **Combined** — MULTI/EXEC inside a pipeline: atomic AND efficient (one round trip for the whole transaction)
- **Choosing between them** — use pipelining for independent bulk writes; use MULTI/EXEC when ordering and exclusivity matter

## Real World Context

You are processing an end-of-day report: you need to read 1,000 counters (independent, use pipelining) and then atomically reset them all to zero (atomic, use MULTI/EXEC). These are two separate operations with different requirements.

## Deep Dive

Pipelining without atomicity:

```python
# Fast but NOT atomic — other clients can interleave
with r.pipeline(transaction=False) as pipe:
    pipe.incr('page_views')
    pipe.incr('unique_visitors')
    results = pipe.execute()
# If another client writes between these two INCRs, it's allowed
```

MULTI/EXEC for atomicity (also pipelined by default in redis-py):

```python
# Atomic AND pipelined — one round trip, no interleaving
with r.pipeline(transaction=True) as pipe:  # transaction=True = MULTI/EXEC
    pipe.incr('page_views')
    pipe.incr('unique_visitors')
    results = pipe.execute()
# MULTI ... INCR page_views ... INCR unique_visitors ... EXEC sent in one batch
```

Comparison table:

| Property | Pipeline (no tx) | MULTI/EXEC | Pipeline + MULTI/EXEC |
|---|---|---|---|
| Round trips | 1 | 1* | 1 |
| Atomic | No | Yes | Yes |
| Other clients interleave | Yes | No | No |
| WATCH compatible | No | Yes | Yes |

*redis-py sends MULTI/EXEC commands as a pipelined batch automatically.

## Common Pitfalls

1. **Using pipelining when atomicity is required.** If you need both page_views and unique_visitors to be incremented together without any other client seeing them half-updated, pipelining alone is insufficient.
2. **Always using MULTI/EXEC when pipelining would suffice.** MULTI/EXEC adds slight overhead from the additional commands. For bulk independent writes, plain pipelining is more efficient.
3. **Thinking pipeline + MULTI/EXEC costs two round trips.** In redis-py, MULTI/EXEC is always pipelined — MULTI, commands, and EXEC are sent in one batch.

## Best Practices

1. Use pipeline(transaction=False) for bulk independent operations.
2. Use pipeline(transaction=True) (or MULTI/EXEC explicitly) when atomicity is required.
3. Never use MULTI/EXEC for operations that depend on a watched key's value being stable — combine with WATCH.

## Summary

- Pipelining reduces RTT; MULTI/EXEC provides atomicity; combining them gives both
- Pipelining alone is NOT atomic — other clients can interleave
- MULTI/EXEC ensures no interleaving but does not make commands faster on the server
- redis-py's transaction=True pipeline automatically wraps commands in MULTI/EXEC
- Choose the tool that matches your correctness requirement, then optimize for performance

## Code Examples

**Pipelining with and without MULTI/EXEC atomicity**

```python
import redis

r = redis.Redis()

# Not atomic — fast bulk writes
with r.pipeline(transaction=False) as pipe:
    pipe.incr('page_views')
    pipe.incr('unique_visitors')
    results = pipe.execute()

# Atomic AND pipelined — use when both matter
with r.pipeline(transaction=True) as pipe:
    pipe.incr('page_views')
    pipe.incr('unique_visitors')
    results = pipe.execute()
```


## Resources

- [Redis Pipelining](https://redis.io/docs/latest/develop/use/pipelining/) — Guide to Redis pipelining
- [Redis Transactions](https://redis.io/docs/latest/develop/interact/transactions/) — Official guide to Redis transactions

---

> 📘 *This lesson is part of the [Scripting, Transactions, and Programmability](https://stanza.dev/courses/redis-scripting) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*