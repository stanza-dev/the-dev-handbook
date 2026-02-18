---
source_course: "redis-scripting"
source_lesson: "redis-scripting-watch-patterns"
---

# WATCH Patterns and Comparison

Learn common patterns using WATCH and when to choose alternatives.

## Inventory Reservation Pattern

```python
def reserve_inventory(product_id, quantity, user_id):
    """
    Reserve inventory for a user.
    Only succeeds if enough stock available.
    """
    stock_key = f'product:{product_id}:stock'
    reservation_key = f'user:{user_id}:cart:{product_id}'
    
    for _ in range(5):
        try:
            r.watch(stock_key)
            
            stock = int(r.get(stock_key) or 0)
            existing = int(r.get(reservation_key) or 0)
            
            if stock < quantity:
                r.unwatch()
                return {'error': 'Insufficient stock'}
            
            pipe = r.pipeline()
            pipe.multi()
            pipe.decrby(stock_key, quantity)
            pipe.set(reservation_key, existing + quantity, ex=900)  # 15min
            pipe.execute()
            
            return {'reserved': quantity, 'expires_in': 900}
            
        except redis.WatchError:
            continue
    
    return {'error': 'Could not complete reservation'}
```

## Compare-and-Swap (CAS)

```python
def compare_and_swap(key, expected, new_value):
    """Set key to new_value only if current value equals expected"""
    for _ in range(5):
        try:
            r.watch(key)
            current = r.get(key)
            
            if current != expected:
                r.unwatch()
                return False
            
            pipe = r.pipeline()
            pipe.multi()
            pipe.set(key, new_value)
            pipe.execute()
            return True
            
        except redis.WatchError:
            continue
    
    return False
```

## WATCH vs Lua Scripts

```
┌─────────────────────────────────────────────────────────────┐
│                WATCH               Lua Script               │
├─────────────────────────────────────────────────────────────┤
│ Retries on contention     │  No retries needed (atomic)    │
│ Multiple round trips      │  Single round trip             │
│ Logic in client           │  Logic on server               │
│ Easier debugging          │  Harder debugging              │
│ No new language           │  Requires Lua                  │
│ Works everywhere          │  Script loading required       │
└─────────────────────────────────────────────────────────────┘
```

### Same Logic: WATCH vs Lua

```python
# WATCH version
def increment_if_less_than_watch(key, max_value):
    for _ in range(5):
        try:
            r.watch(key)
            val = int(r.get(key) or 0)
            if val >= max_value:
                r.unwatch()
                return None
            pipe = r.pipeline()
            pipe.multi()
            pipe.incr(key)
            return pipe.execute()[0]
        except redis.WatchError:
            continue
    return None

# Lua version - no retries needed
LUA_SCRIPT = """
local val = tonumber(redis.call('GET', KEYS[1])) or 0
if val >= tonumber(ARGV[1]) then
    return nil
end
return redis.call('INCR', KEYS[1])
"""

def increment_if_less_than_lua(key, max_value):
    return r.eval(LUA_SCRIPT, 1, key, max_value)
```

## When to Use Each

| Scenario | Best Choice |
|----------|-------------|
| Low contention (< 10 writes/sec) | WATCH |
| High contention | Lua script |
| Complex multi-key logic | Lua script |
| Simple check-and-set | WATCH |
| Need to read during transaction | Lua script |
| Don't want Lua dependency | WATCH |

📖 [Transactions](https://redis.io/docs/latest/develop/interact/transactions/)

## Resources

- [Redis Transactions](https://redis.io/docs/latest/develop/interact/transactions/) — Transactions and WATCH documentation

---

> 📘 *This lesson is part of the [Scripting, Transactions, and Programmability](https://stanza.dev/courses/redis-scripting) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*