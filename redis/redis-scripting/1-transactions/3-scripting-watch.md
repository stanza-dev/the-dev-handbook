---
source_course: "redis-scripting"
source_lesson: "redis-scripting-watch"
---

# Optimistic Locking with WATCH

## Introduction

WATCH provides optimistic locking for Redis transactions. It lets you read a key's value and then conditionally execute a transaction — only if that key was not modified by another client in the meantime. This is the standard pattern for safe read-modify-write operations in Redis.

## Key Concepts

- **WATCH key [key ...]** — marks keys to monitor; if any change before EXEC, the transaction is aborted
- **Optimistic locking** — assumes conflicts are rare; proceeds optimistically and detects conflicts at commit time
- **EXEC returning nil** — signals that a watched key changed and the transaction was aborted
- **UNWATCH** — cancels all active watches without discarding any queued commands
- **Auto-unwatch** — EXEC and DISCARD both automatically cancel all watches

## Real World Context

You are building a ticket booking system. You read the available seat count, decide whether to allow the booking, then decrement the count. Without WATCH, two clients could both read the same count, both decide to book, and both decrement — overselling by one. WATCH detects if another client changed the count before you EXEC.

## Deep Dive

The complete WATCH pattern is:

```bash
WATCH seats:concert-101
GET seats:concert-101
# Returns: 5
# Client decides booking is valid
MULTI
DECR seats:concert-101
EXEC
# If another client changed seats:concert-101 between WATCH and EXEC:
# EXEC returns (nil) — transaction aborted
# Otherwise EXEC returns: 1) (integer) 4
```

When EXEC returns nil, your application should retry the whole sequence — re-WATCH, re-read, re-decide, and re-EXEC. This is why WATCH operations need a retry loop.

You can watch multiple keys at once:

```bash
WATCH account:1001:balance account:1002:balance
MULTI
DECRBY account:1001:balance 100
INCRBY account:1002:balance 100
EXEC
```

## Common Pitfalls

1. **Forgetting to retry on nil.** If EXEC returns nil, the application must loop back and try again; a single attempt is not sufficient for correctness.
2. **Watching too many keys.** Every additional watched key is another opportunity for the transaction to abort, increasing retry rates under high concurrency.
3. **Calling WATCH inside MULTI.** WATCH must be called before MULTI — it monitors keys during the gap between WATCH and EXEC.

## Best Practices

1. Always implement a retry loop with a bounded number of attempts and a backoff for WATCH-based transactions.
2. Consider using Lua scripting for high-contention scenarios — Lua scripts execute atomically and never abort due to concurrent writes.
3. Call UNWATCH explicitly if you decide to abort early and exit the retry loop without calling EXEC or DISCARD.

## Summary

- WATCH sets up optimistic locking before a MULTI/EXEC transaction
- If any watched key changes between WATCH and EXEC, EXEC returns nil
- The application must detect nil and retry
- EXEC and DISCARD automatically cancel all active watches
- UNWATCH cancels watches manually without discarding queued commands
- High-contention scenarios may benefit from Lua scripting instead

## Code Examples

**WATCH/MULTI/EXEC optimistic locking pattern**

```bash
WATCH seats:concert-101
GET seats:concert-101
# => 5
MULTI
DECR seats:concert-101
EXEC
# If key changed: (nil)
# If key unchanged: 1) (integer) 4
```


## Resources

- [WATCH Command](https://redis.io/commands/watch/) — WATCH command reference and optimistic locking guide

---

> 📘 *This lesson is part of the [Scripting, Transactions, and Programmability](https://stanza.dev/courses/redis-scripting) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*