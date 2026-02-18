---
source_course: "redis-scripting"
source_lesson: "redis-scripting-watch"
---

# Optimistic Locking with WATCH

WATCH provides optimistic locking - it allows transactions to fail if watched keys have been modified.

## How WATCH Works

```redis
# Watch a key
WATCH user:1001:balance

# Read current value
GET user:1001:balance
# Returns: "100"

# Start transaction based on read value
MULTI
DECRBY user:1001:balance 50
EXEC
```

If another client modifies `user:1001:balance` between WATCH and EXEC:

```redis
EXEC
# Returns: (nil) - Transaction aborted!
```

## WATCH Flow

```
┌─────────────────────────────────────────────────────────────┐
│  Client A:                    Client B:                     │
│  WATCH balance               │                              │
│  GET balance → 100           │                              │
│                              │ SET balance 50 (modify!)     │
│  MULTI                       │                              │
│  DECRBY balance 25           │                              │
│  EXEC                        │                              │
│  → (nil) Transaction failed! │                              │
└─────────────────────────────────────────────────────────────┘
```

## Check-and-Set Pattern

```python
import redis

r = redis.Redis(decode_responses=True)

def safe_transfer(from_user, to_user, amount, max_retries=5):
    """Transfer funds with optimistic locking"""
    from_key = f'user:{from_user}:balance'
    to_key = f'user:{to_user}:balance'
    
    for attempt in range(max_retries):
        try:
            # Watch the source account
            r.watch(from_key)
            
            # Check balance
            balance = int(r.get(from_key) or 0)
            if balance < amount:
                r.unwatch()
                return False, "Insufficient funds"
            
            # Execute transaction
            pipe = r.pipeline()
            pipe.multi()
            pipe.decrby(from_key, amount)
            pipe.incrby(to_key, amount)
            pipe.execute()
            
            return True, "Transfer successful"
            
        except redis.WatchError:
            # Key was modified, retry
            continue
    
    return False, "Max retries exceeded"
```

## Watching Multiple Keys

```redis
# Watch multiple keys
WATCH key1 key2 key3

# Transaction fails if ANY watched key is modified
MULTI
SET key1 "new_value"
EXEC
```

## UNWATCH

```redis
# Cancel all watches
UNWATCH

# Useful when you decide not to proceed with transaction
WATCH key1
GET key1
# Business logic decides not to proceed
UNWATCH  # Release the watch
```

## Practical Example: Inventory Management

```python
def reserve_item(user_id, item_id, quantity):
    """Reserve inventory atomically"""
    inventory_key = f'item:{item_id}:stock'
    reservation_key = f'user:{user_id}:reservations'
    
    for _ in range(5):  # Max retries
        try:
            r.watch(inventory_key)
            
            stock = int(r.get(inventory_key) or 0)
            if stock < quantity:
                r.unwatch()
                return False, "Insufficient stock"
            
            pipe = r.pipeline()
            pipe.multi()
            pipe.decrby(inventory_key, quantity)
            pipe.hincrby(reservation_key, item_id, quantity)
            pipe.execute()
            
            return True, "Reserved"
            
        except redis.WatchError:
            continue
    
    return False, "Could not acquire lock"
```

## When to Use WATCH vs Lua Scripts

| Scenario | Recommendation |
|----------|----------------|
| Simple check-and-set | WATCH |
| Complex logic | Lua script |
| Need to read during transaction | Lua script |
| High contention | Lua script (faster) |

📖 [WATCH Command](https://redis.io/docs/latest/commands/watch/)

---

> 📘 *This lesson is part of the [Scripting, Transactions, and Programmability](https://stanza.dev/courses/redis-scripting) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*