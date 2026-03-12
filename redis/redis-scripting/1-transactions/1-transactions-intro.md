---
source_course: "redis-scripting"
source_lesson: "redis-scripting-transactions-intro"
---

# Understanding Redis Transactions

## Introduction

Redis transactions let you execute a sequence of commands as an uninterrupted block — no other client's commands will interleave during execution. This is essential when multiple related writes must be applied together, such as debiting one account and crediting another.

## Key Concepts

- **MULTI** — begins a transaction block; subsequent commands are queued, not executed
- **EXEC** — sends all queued commands to Redis, which executes them sequentially without interruption from other clients
- **DISCARD** — aborts the transaction and clears the command queue
- **Atomicity in Redis** — means commands execute without interleaving, NOT that they roll back on failure; each command in the queue still succeeds or fails independently

## Real World Context

Imagine a wallet service where a user purchases an item: you must debit the user's balance and record the purchase in one step. Without a transaction, another request could read the balance between the debit and the record, seeing an inconsistent state.

## Deep Dive

When a client sends `MULTI`, Redis switches that connection into transaction mode. Every subsequent command returns `QUEUED` instead of executing:

```bash
MULTI
SET user:1001:balance 100
INCRBY user:1001:balance 50
DECRBY user:1001:balance 25
EXEC
```

When `EXEC` is called, Redis runs all queued commands back-to-back. No other client can inject a command between them. The response is an array of results:

```bash
# Response from EXEC:
1) OK
2) (integer) 150
3) (integer) 125
```

Note that if one command inside the transaction has a runtime error (e.g., calling INCR on a string value), that single command fails but the remaining commands still execute. Redis does not roll back. Use `DISCARD` before `EXEC` to cancel the whole transaction if you detect a problem:

```bash
MULTI
SET key "value"
DISCARD
# Queue is cleared, transaction aborted
```

## Common Pitfalls

1. **Assuming rollback exists.** Redis transactions do not roll back on runtime errors. If `INCR` is called on a string key inside a transaction, INCR fails but the other commands still run. Design your logic accordingly.
2. **Calling EXEC after a syntax error.** If a command in the queue has a syntax error (unknown command name), Redis will abort the entire transaction on EXEC and return an error. This is the only case where no commands execute.
3. **Forgetting DISCARD on error paths.** If your application detects a precondition failure and wants to abort, always call DISCARD; otherwise the connection stays in transaction mode.

## Best Practices

1. Keep transactions short — long queues delay EXEC and increase the window where external state can change.
2. Validate all inputs before calling MULTI so you can avoid entering a transaction you'll just DISCARD.
3. Always wrap transaction logic in a try/finally block (in application code) to ensure DISCARD is called if an exception occurs before EXEC.

## Summary

- MULTI starts a transaction; commands after it are QUEUED
- EXEC executes all queued commands atomically (no interleaving)
- Atomicity here means no interruption — NOT automatic rollback
- Runtime errors inside a transaction affect only that command; others execute
- Syntax errors abort the entire transaction on EXEC
- DISCARD clears the queue and exits transaction mode

## Code Examples

**Basic MULTI/EXEC transaction**

```bash
MULTI
SET user:1001:balance 100
INCRBY user:1001:balance 50
DECRBY user:1001:balance 25
EXEC
# Response:
# 1) OK
# 2) (integer) 150
# 3) (integer) 125
```

**Aborting a transaction with DISCARD**

```bash
MULTI
SET key "value"
DISCARD
# Queue cleared, no commands executed
```


## Resources

- [Redis Transactions](https://redis.io/docs/latest/develop/interact/transactions/) — Official guide to Redis transactions

---

> 📘 *This lesson is part of the [Scripting, Transactions, and Programmability](https://stanza.dev/courses/redis-scripting) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*