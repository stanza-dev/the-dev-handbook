---
source_course: "redis-scripting"
source_lesson: "redis-scripting-transaction-errors"
---

# Error Handling Inside Transactions

## Introduction

Redis distinguishes between two kinds of errors inside a transaction: syntax errors detected at queue time, and runtime errors that occur during EXEC. Understanding this distinction is critical — the behavior is different from most databases and can surprise developers.

## Key Concepts

- **Syntax error** — an unknown or malformed command that Redis rejects immediately when queued; causes the entire transaction to be aborted on EXEC
- **Runtime error** — a command that is syntactically valid but fails during execution (e.g., INCR on a string); only that command fails; others execute
- **No rollback** — Redis has no ROLLBACK command; commands that already executed are permanent
- **EXECABORT** — the error returned by EXEC when a syntax error was queued

## Real World Context

A payment service queues five commands in a transaction. One command uses the wrong type on a key. With SQL, the whole transaction would roll back. With Redis, four commands succeed and only the erroneous one fails — your application code must handle partial success.

## Deep Dive

Here is what happens with a syntax error (typo in command name):

```bash
MULTI
SET counter 1
INKR counter
# (error) ERR unknown command 'INKR'
# Redis marks transaction as errored
EXEC
# (error) EXECABORT Transaction discarded because of previous errors.
```

Notice that SET counter 1 also did NOT execute — the whole queue was discarded.

Now compare with a runtime error (wrong type):

```bash
SET mystring "hello"
MULTI
SET other "world"
INCR mystring
SET third "done"
EXEC
# 1) OK           <- SET other succeeded
# 2) (error) WRONGTYPE ...
# 3) OK           <- SET third succeeded
```

The INCR failed because mystring holds a string, not an integer. But both SET commands executed successfully. There is no rollback.

## Common Pitfalls

1. **Assuming a runtime error aborts the transaction.** It does not. Commands before and after the failing command still execute. Validate types before entering MULTI.
2. **Confusing syntax errors with runtime errors.** A typo in a command name aborts everything; a wrong-type error only fails that one command.
3. **Not checking EXEC response elements.** Each element in the EXEC response array can be either a result or an error. Always iterate and check for error objects in your client code.

## Best Practices

1. Verify key types with TYPE command before entering a transaction if your logic depends on them.
2. In application code, check every entry in the EXEC response array for errors, not just the array itself.
3. Use Lua scripting instead of MULTI/EXEC when you need true abort-on-error semantics — a Lua script stops executing on redis.call() errors.

## Summary

- Syntax errors (unknown commands) abort the entire transaction on EXEC
- Runtime errors (wrong type, etc.) only fail that single command
- The remaining commands in the queue still execute after a runtime error
- Redis has no ROLLBACK — partial execution is possible
- Always check each element of the EXEC response for errors in client code

## Code Examples

**Syntax error aborts the entire transaction**

```bash
# Syntax error example — whole transaction aborted
MULTI
SET counter 1
INKR counter
# (error) ERR unknown command 'INKR'
EXEC
# (error) EXECABORT Transaction discarded because of previous errors.
```

**Runtime error fails only that command; others execute**

```bash
# Runtime error example — other commands still execute
SET mystring "hello"
MULTI
SET other "world"
INCR mystring
SET third "done"
EXEC
# 1) OK
# 2) (error) WRONGTYPE Operation against a key holding the wrong kind of value
# 3) OK
```


## Resources

- [Redis Transactions](https://redis.io/docs/latest/develop/interact/transactions/) — Official guide covering error handling in transactions

---

> 📘 *This lesson is part of the [Scripting, Transactions, and Programmability](https://stanza.dev/courses/redis-scripting) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*