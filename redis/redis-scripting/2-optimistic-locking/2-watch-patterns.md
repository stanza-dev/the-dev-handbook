---
source_course: "redis-scripting"
source_lesson: "redis-scripting-watch-patterns"
---

# WATCH Patterns and Comparisons

## Introduction

WATCH is versatile but not always the right tool. This lesson covers the most useful WATCH patterns — counter increments, conditional updates, and inventory reservation — and explains when to use WATCH versus Lua scripting.

## Key Concepts

- **Counter increment with floor/ceiling** — read, bound-check, increment; use WATCH to prevent race conditions
- **Conditional update** — only write if the current value meets a condition
- **Inventory reservation** — reserve an item only if stock > 0, decrement atomically
- **WATCH vs Lua** — WATCH retries on conflict; Lua executes atomically and never retries

## Real World Context

An e-commerce site needs to reserve the last item in stock. Multiple users click "Buy" simultaneously. WATCH ensures only one succeeds; the others get a "sold out" response.

## Deep Dive

Inventory reservation pattern:

```python
def reserve_inventory(r, product_id, quantity):
    key = f"inventory:{product_id}"
    for attempt in range(5):
        with r.pipeline() as pipe:
            try:
                pipe.watch(key)
                current = int(pipe.get(key) or 0)
                if current < quantity:
                    pipe.unwatch()
                    return False  # Insufficient stock
                pipe.multi()
                pipe.decrby(key, quantity)
                pipe.execute()
                return True
            except redis.WatchError:
                continue
    return False  # Exhausted retries
```

The equivalent Lua script (simpler under high contention):

```lua
-- reserve_inventory.lua
-- KEYS[1] = inventory key, ARGV[1] = quantity
local current = tonumber(redis.call('GET', KEYS[1])) or 0
local qty = tonumber(ARGV[1])
if current < qty then
    return 0
end
redis.call('DECRBY', KEYS[1], qty)
return 1
```

The Lua version runs atomically in a single EVAL call — no retry loop needed.

When to choose WATCH:
- The conditional logic depends on data that cannot be computed server-side
- You need to read from the database or external service before deciding
- Low-to-moderate contention where retries are rare

When to choose Lua:
- High contention where retries would be frequent
- All logic can be expressed in Lua with Redis calls
- You want simpler client code

## Common Pitfalls

1. **Using WATCH for non-contested keys.** If only one client writes a key, WATCH adds overhead with no benefit. Use a simple SET or INCR instead.
2. **Forgetting to handle the "sold out" case** in the retry loop. After exhausting retries, return a meaningful error, not a silent failure.
3. **Mixing WATCH and Lua.** Do not use WATCH inside a Lua script — Lua scripts are already atomic and WATCH has no effect inside them.

## Best Practices

1. Prefer Lua for inventory/counter patterns — it is simpler and scales better under contention.
2. Use WATCH when the decision depends on external data that Redis cannot compute.
3. Always set a maximum retry count and surface a conflict error to the caller when exhausted.

## Summary

- WATCH patterns: conditional update, counter with bounds, inventory reservation
- Lua scripts handle the same patterns atomically without client-side retry loops
- Choose WATCH when external data is needed for the decision
- Choose Lua when all logic can run server-side and contention is high
- Always cap retry loops to prevent infinite spinning

## Code Examples

**Inventory reservation using WATCH retry loop**

```python
def reserve_inventory(r, product_id, quantity):
    key = f"inventory:{product_id}"
    for attempt in range(5):
        with r.pipeline() as pipe:
            try:
                pipe.watch(key)
                current = int(pipe.get(key) or 0)
                if current < quantity:
                    pipe.unwatch()
                    return False
                pipe.multi()
                pipe.decrby(key, quantity)
                pipe.execute()
                return True
            except redis.WatchError:
                continue
    return False
```

**Same pattern as a Lua script (no retry needed)**

```lua
-- KEYS[1] = inventory key, ARGV[1] = quantity
local current = tonumber(redis.call('GET', KEYS[1])) or 0
local qty = tonumber(ARGV[1])
if current < qty then
    return 0
end
redis.call('DECRBY', KEYS[1], qty)
return 1
```


## Resources

- [Redis Transactions](https://redis.io/docs/latest/develop/interact/transactions/) — Transactions and WATCH documentation

---

> 📘 *This lesson is part of the [Scripting, Transactions, and Programmability](https://stanza.dev/courses/redis-scripting) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*