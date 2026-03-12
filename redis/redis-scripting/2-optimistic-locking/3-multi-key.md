---
source_course: "redis-scripting"
source_lesson: "redis-scripting-watch-multi-key"
---

# Watching Multiple Keys

## Introduction

WATCH accepts multiple keys in a single call, allowing you to build transactions that depend on the state of several related keys simultaneously. This enables complex conditional workflows — but also increases the chance of transaction aborts under concurrency.

## Key Concepts

- **WATCH key [key ...]** — all listed keys are watched simultaneously; any modification to any of them causes EXEC to return nil
- **Additive watches** — calling WATCH multiple times adds to the set of watched keys (does not replace)
- **Abort on any change** — even one changed key among many aborts the whole transaction
- **UNWATCH resets all** — UNWATCH cancels every watched key on the connection at once

## Real World Context

A fund transfer must read both the sender's balance and the receiver's balance before writing. Both keys must be watched: if either account is modified by another transaction between your read and your EXEC, the transfer must be retried from scratch.

## Deep Dive

Watching two keys simultaneously:

```bash
WATCH account:sender:balance account:receiver:balance
GET account:sender:balance
# => 500
GET account:receiver:balance
# => 100
MULTI
DECRBY account:sender:balance 100
INCRBY account:receiver:balance 100
EXEC
# If EITHER key changed: (nil)
# If both unchanged: array of results
```

Calling WATCH multiple times is additive:

```bash
WATCH key1
WATCH key2
# Now both key1 and key2 are watched
# Equivalent to: WATCH key1 key2
```

The Python transfer example:

```python
def transfer(r, sender_id, receiver_id, amount):
    sender_key = f"account:{sender_id}:balance"
    receiver_key = f"account:{receiver_id}:balance"
    for _ in range(5):
        with r.pipeline() as pipe:
            try:
                pipe.watch(sender_key, receiver_key)
                sender_bal = int(pipe.get(sender_key) or 0)
                if sender_bal < amount:
                    pipe.unwatch()
                    return False  # Insufficient funds
                pipe.multi()
                pipe.decrby(sender_key, amount)
                pipe.incrby(receiver_key, amount)
                pipe.execute()
                return True
            except Exception:
                continue
    return False
```

## Common Pitfalls

1. **Watching too many keys increases abort probability.** Each additional watched key multiplies the chance of abort under concurrency.
2. **Not UNWATCH-ing before early exit.** If a precondition fails before MULTI, call UNWATCH to release the watches explicitly.
3. **Expecting WATCH to lock keys.** WATCH is optimistic — it does not prevent other clients from modifying watched keys; it only detects changes.

## Best Practices

1. Minimize the number of watched keys to the strict minimum needed for correctness.
2. Consider Lua scripting for multi-key atomic operations to avoid the abort risk entirely.
3. Always call UNWATCH when exiting the retry loop early without completing a transaction.

## Summary

- WATCH accepts multiple keys; any modification to any watched key aborts EXEC
- Multiple WATCH calls are additive — they do not replace previous watches
- UNWATCH cancels all watches on the connection at once
- Watching more keys increases the probability of transaction abort under concurrency
- For multi-key atomic operations under high contention, Lua scripting is often simpler

## Code Examples

**Watching two keys for an atomic fund transfer**

```bash
WATCH account:sender:balance account:receiver:balance
GET account:sender:balance
GET account:receiver:balance
MULTI
DECRBY account:sender:balance 100
INCRBY account:receiver:balance 100
EXEC
# If either key changed: (nil)
# If both unchanged: 1) (integer) 400  2) (integer) 200
```

**Multi-key WATCH transfer in Python with retry loop**

```python
def transfer(r, sender_id, receiver_id, amount):
    sender_key = f"account:{sender_id}:balance"
    receiver_key = f"account:{receiver_id}:balance"
    for _ in range(5):
        with r.pipeline() as pipe:
            try:
                pipe.watch(sender_key, receiver_key)
                sender_bal = int(pipe.get(sender_key) or 0)
                if sender_bal < amount:
                    pipe.unwatch()
                    return False
                pipe.multi()
                pipe.decrby(sender_key, amount)
                pipe.incrby(receiver_key, amount)
                pipe.execute()
                return True
            except Exception:
                continue
    return False
```


## Resources

- [WATCH Command](https://redis.io/commands/watch/) — WATCH command reference including multi-key usage

---

> 📘 *This lesson is part of the [Scripting, Transactions, and Programmability](https://stanza.dev/courses/redis-scripting) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*