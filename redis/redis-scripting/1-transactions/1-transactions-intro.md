---
source_course: "redis-scripting"
source_lesson: "redis-scripting-transactions-intro"
---

# Understanding Redis Transactions

Redis transactions allow you to execute multiple commands atomically - either all commands execute, or none do.

## MULTI/EXEC Basics

```redis
# Start transaction
MULTI

# Queue commands (not executed yet)
SET user:1001:balance 100
INCRBY user:1001:balance 50
DECRBY user:1001:balance 25

# Execute all commands atomically
EXEC
# Returns:
# 1) OK
# 2) (integer) 150
# 3) (integer) 125
```

## How Transactions Work

```
┌─────────────────────────────────────────────────────────────┐
│  1. MULTI                                                   │
│     → Redis enters transaction mode                         │
│     → Returns: OK                                           │
│                                                             │
│  2. Commands (SET, INCR, etc.)                             │
│     → Commands are QUEUED, not executed                    │
│     → Returns: QUEUED                                       │
│                                                             │
│  3. EXEC                                                    │
│     → All queued commands execute atomically               │
│     → No other client commands can interleave              │
│     → Returns: Array of results                            │
└─────────────────────────────────────────────────────────────┘
```

## DISCARD: Cancel Transaction

```redis
MULTI
SET key1 "value1"
SET key2 "value2"
DISCARD   # Cancel transaction
# All queued commands are discarded
```

## Important Characteristics

### All or Nothing Execution (Sort Of)

```redis
MULTI
SET key1 "value1"
INCR key1           # This will fail (key1 is not a number)
SET key2 "value2"
EXEC
# Returns:
# 1) OK
# 2) (error) ERR value is not an integer
# 3) OK
```

**Important**: Redis transactions are NOT like SQL transactions!
- Commands execute sequentially
- If one fails, others STILL execute
- No rollback on error

### Atomicity Guarantee

```
┌─────────────────────────────────────────────────────────────┐
│  During EXEC, no other commands can run between your       │
│  transaction commands. It's atomic in terms of isolation.  │
│                                                             │
│  Client A:  MULTI → SET a 1 → SET b 2 → EXEC              │
│  Client B:  GET a (waits until Client A's EXEC completes) │
└─────────────────────────────────────────────────────────────┘
```

## Practical Example: Transfer Money

```redis
# Transfer $50 from user:1001 to user:1002
MULTI
DECRBY user:1001:balance 50
INCRBY user:1002:balance 50
EXEC
```

**Problem**: What if user:1001 doesn't have $50? The DECRBY still executes!

Solution: Use WATCH (next lesson) or Lua scripts.

## Transaction with Error Handling

```python
import redis

r = redis.Redis()

def transfer_funds(from_user, to_user, amount):
    pipe = r.pipeline()
    try:
        pipe.multi()
        pipe.decrby(f'user:{from_user}:balance', amount)
        pipe.incrby(f'user:{to_user}:balance', amount)
        results = pipe.execute()
        return True
    except redis.exceptions.ResponseError as e:
        pipe.reset()
        return False
```

## Commands Not Allowed in Transactions

- WATCH, UNWATCH, MULTI (can't nest transactions)
- Commands that block (BLPOP, BRPOP, etc.)
- SUBSCRIBE, UNSUBSCRIBE

📖 [Redis Transactions](https://redis.io/docs/latest/develop/interact/transactions/)

## Resources

- [Redis Transactions](https://redis.io/docs/latest/develop/interact/transactions/) — Official guide to Redis transactions

---

> 📘 *This lesson is part of the [Scripting, Transactions, and Programmability](https://stanza.dev/courses/redis-scripting) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*