---
source_course: "redis-scripting"
source_lesson: "redis-scripting-watch-deep"
---

# Check-and-Set with WATCH

WATCH enables optimistic locking - transactions fail if watched keys change before EXEC.

## The Check-and-Set Problem

Without WATCH, you can't safely read and conditionally modify:

```
┌─────────────────────────────────────────────────────────────┐
│  Thread A:                    Thread B:                     │
│  balance = GET balance        │                             │
│  (balance = 100)              │                             │
│                               │ DECRBY balance 60           │
│                               │ (balance = 40)              │
│  if balance >= 50:            │                             │
│    DECRBY balance 50          │                             │
│  (balance = -10!)  ← RACE CONDITION!                        │
└─────────────────────────────────────────────────────────────┘
```

## WATCH Solves This

```redis
WATCH balance
GET balance            # 100

# Another client: DECRBY balance 60 (now 40)

MULTI
DECRBY balance 50
EXEC
# Returns: (nil) - Transaction aborted!
```

## Complete Implementation Pattern

```python
import redis

r = redis.Redis(decode_responses=True)

def atomic_withdraw(account_key, amount, max_retries=5):
    """
    Withdraw amount if sufficient balance.
    Uses optimistic locking with retry.
    """
    for attempt in range(max_retries):
        try:
            # Start watching
            r.watch(account_key)
            
            # Read current balance
            balance = int(r.get(account_key) or 0)
            
            # Check condition
            if balance < amount:
                r.unwatch()
                return {'success': False, 'error': 'Insufficient funds'}
            
            # Execute conditional update
            pipe = r.pipeline()
            pipe.multi()
            pipe.decrby(account_key, amount)
            result = pipe.execute()
            
            return {
                'success': True, 
                'new_balance': result[0]
            }
            
        except redis.WatchError:
            # Key was modified by another client, retry
            continue
    
    return {'success': False, 'error': 'Max retries exceeded'}
```

## Multi-Key WATCH

```python
def transfer(from_account, to_account, amount):
    """Transfer between accounts atomically"""
    for _ in range(5):
        try:
            # Watch both accounts
            r.watch(from_account, to_account)
            
            from_balance = int(r.get(from_account) or 0)
            
            if from_balance < amount:
                r.unwatch()
                return False
            
            pipe = r.pipeline()
            pipe.multi()
            pipe.decrby(from_account, amount)
            pipe.incrby(to_account, amount)
            pipe.execute()
            return True
            
        except redis.WatchError:
            continue
    
    return False
```

## WATCH Best Practices

```
┌─────────────────────────────────────────────────────────────┐
│  1. Keep the window small between WATCH and EXEC           │
│  2. Don't do slow operations (DB queries) inside           │
│  3. Always UNWATCH on early exit                           │
│  4. Implement retry with backoff for high contention       │
│  5. Consider Lua scripts for very high contention          │
└─────────────────────────────────────────────────────────────┘
```

```python
def watched_operation_with_backoff(key, max_retries=5):
    for attempt in range(max_retries):
        try:
            r.watch(key)
            # ... operation ...
            pipe.execute()
            return True
        except redis.WatchError:
            # Exponential backoff with jitter
            delay = (2 ** attempt) * 0.001 + random.uniform(0, 0.001)
            time.sleep(delay)
    return False
```

📖 [WATCH Command](https://redis.io/docs/latest/commands/watch/)

## Resources

- [WATCH Command](https://redis.io/docs/latest/commands/watch/) — WATCH command reference

---

> 📘 *This lesson is part of the [Scripting, Transactions, and Programmability](https://stanza.dev/courses/redis-scripting) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*