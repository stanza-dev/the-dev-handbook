---
source_course: "redis-scripting"
source_lesson: "redis-scripting-atomic-rmw"
---

# Atomic Read-Modify-Write

## Introduction

Lua scripts excel at read-modify-write (RMW) operations that must be atomic. The pattern is: read current state, compute new state, write new state — all without any other client being able to observe intermediate state or modify the keys between the read and the write.

## Key Concepts

- **Read-Modify-Write (RMW)** — read a value, compute a new value based on it, write back atomically
- **Atomic Lua** — the entire script runs as a single Redis command; no other commands execute in parallel
- **Why not WATCH** — WATCH retries when conflicts occur; Lua scripts never conflict because they run atomically
- **Appropriate use cases** — counters with bounds, conditional increments, score updates, inventory management

## Real World Context

A leaderboard service must update a player's score only if the new score is higher than the current one. With separate GET + SET commands, two concurrent updates could both read the old score and both write, potentially losing the higher value. A Lua script performs this check-and-update atomically.

## Deep Dive

Conditional update — only update if new value is greater:

```lua
-- update_if_greater.lua
-- KEYS[1] = score key, ARGV[1] = new score
local current = tonumber(redis.call('GET', KEYS[1])) or 0
local new_score = tonumber(ARGV[1])
if new_score > current then
    redis.call('SET', KEYS[1], new_score)
    return new_score
end
return current
```

Bounded counter — increment only if below ceiling:

```lua
-- bounded_incr.lua
-- KEYS[1] = counter key, ARGV[1] = ceiling
local current = tonumber(redis.call('GET', KEYS[1])) or 0
local ceiling = tonumber(ARGV[1])
if current >= ceiling then
    return -1  -- cannot increment
end
return redis.call('INCR', KEYS[1])
```

Execute with:

```bash
EVAL "$(cat bounded_incr.lua)" 1 counter:downloads 100
```

Why Lua beats WATCH for these patterns:
- No retry loop needed
- No possibility of starvation under high contention
- Cleaner client code
- Guaranteed to succeed on first attempt

## Common Pitfalls

1. **Long-running scripts blocking Redis.** If a script iterates large sets or sleeps, it blocks all other clients. Redis will kill scripts exceeding lua-time-limit (default 5 seconds).
2. **Modifying keys not declared in KEYS array.** In Redis Cluster, this causes a CROSSSLOT error. Always declare all keys.
3. **Not handling nil returns from GET.** If a key does not exist, GET returns nil in Lua. Always apply `or 0` (or a suitable default) when expecting a number.

## Best Practices

1. Replace WATCH retry loops with Lua scripts for all pure server-side conditional updates.
2. Use `tonumber(...) or 0` to safely convert possibly-nil GET results to numbers.
3. Load scripts with SCRIPT LOAD and use EVALSHA in production to save bandwidth.

## Summary

- Lua RMW scripts: GET → compute → SET in one atomic EVAL
- No retry loop needed — scripts cannot be interrupted
- Safer than WATCH for high-contention counters and conditional updates
- Always handle nil GET results with `tonumber(...) or 0`
- Declare all accessed keys in the KEYS array for cluster compatibility

## Code Examples

**Atomic conditional update: write only if new value is greater**

```lua
-- update_if_greater.lua
-- KEYS[1] = score key, ARGV[1] = new score
local current = tonumber(redis.call('GET', KEYS[1])) or 0
local new_score = tonumber(ARGV[1])
if new_score > current then
    redis.call('SET', KEYS[1], new_score)
    return new_score
end
return current
```

**Bounded increment: increment only if below ceiling**

```lua
-- bounded_incr.lua
-- KEYS[1] = counter key, ARGV[1] = ceiling
local current = tonumber(redis.call('GET', KEYS[1])) or 0
local ceiling = tonumber(ARGV[1])
if current >= ceiling then
    return -1
end
return redis.call('INCR', KEYS[1])
```


## Resources

- [Redis Scripting with Lua](https://redis.io/docs/latest/develop/interact/programmability/eval-intro/) — Redis Lua scripting documentation

---

> 📘 *This lesson is part of the [Scripting, Transactions, and Programmability](https://stanza.dev/courses/redis-scripting) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*