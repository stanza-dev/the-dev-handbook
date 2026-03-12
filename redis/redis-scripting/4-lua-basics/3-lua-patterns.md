---
source_course: "redis-scripting"
source_lesson: "redis-scripting-lua-patterns"
---

# Lua Script Patterns

## Introduction

Lua scripts in Redis enable patterns that are impossible or unsafe to implement with plain commands or MULTI/EXEC. This lesson covers the most common production patterns: conditional updates, counter operations with bounds, and caching with computation.

## Key Concepts

- **Conditional update** — check a value then write, atomically; the classic check-and-set done server-side
- **Bounded counter** — increment only if below a ceiling; useful for rate limiting
- **Atomic cache-aside** — check cache, compute if missing, store, return; eliminates the cache stampede
- **redis.pcall()** — protected call; returns error table instead of raising, letting the script handle errors

## Real World Context

A rate limiter needs to: read the current request count, check it against the limit, increment if under limit, and set expiry on first increment — all atomically. A Lua script does this in one EVAL, eliminating race conditions that would exist with separate commands.

## Deep Dive

Pattern 1 — Conditional update (update only if value matches expected):

```lua
-- KEYS[1] = target key, ARGV[1] = expected value, ARGV[2] = new value
local current = redis.call('GET', KEYS[1])
if current == ARGV[1] then
    redis.call('SET', KEYS[1], ARGV[2])
    return 1
end
return 0
```

Pattern 2 — Rate limiter with bounded increment:

```lua
-- KEYS[1] = rate limit key, ARGV[1] = limit, ARGV[2] = window_seconds
local count = redis.call('INCR', KEYS[1])
if count == 1 then
    redis.call('EXPIRE', KEYS[1], ARGV[2])
end
if count > tonumber(ARGV[1]) then
    return 0  -- rate limited
end
return 1  -- allowed
```

Pattern 3 — Error handling with pcall:

```lua
-- Try to increment; if key holds wrong type, return error gracefully
local ok, err = pcall(function()
    return redis.call('INCR', KEYS[1])
end)
-- Note: in Redis Lua, use redis.pcall() directly:
local result = redis.pcall('INCR', KEYS[1])
if type(result) == 'table' and result.err then
    return redis.error_reply('Key has wrong type')
end
return result
```

## Common Pitfalls

1. **Treating redis.call() errors as table returns.** redis.call() raises a Lua error, not a table. Use redis.pcall() if you want to inspect error details without aborting the script.
2. **Blocking scripts with long loops.** A script that loops over thousands of keys blocks Redis for all other clients. Use SCAN-based patterns in application code instead.
3. **Not converting types.** Redis returns strings; Lua comparisons are type-strict. Always use tonumber() for numeric comparisons.

## Best Practices

1. Use redis.pcall() for operations that might fail on wrong key types, then return a meaningful error.
2. Favor short, single-purpose scripts over long scripts that do multiple unrelated things.
3. Document each script with a header comment listing KEYS[n] and ARGV[n] meanings.

## Summary

- Conditional update: GET → compare → SET atomically in one EVAL
- Bounded counter: INCR → compare → conditionally EXPIRE, all in one script
- redis.pcall() returns error as table instead of raising, enabling graceful error handling
- Always use tonumber() when comparing Redis string results as numbers
- Keep scripts focused and fast to minimize the time other clients are blocked

## Code Examples

**Atomic rate limiter using Lua**

```lua
-- Rate limiter: KEYS[1]=limit key, ARGV[1]=limit, ARGV[2]=window_seconds
local count = redis.call('INCR', KEYS[1])
if count == 1 then
    redis.call('EXPIRE', KEYS[1], ARGV[2])
end
if count > tonumber(ARGV[1]) then
    return 0  -- rate limited
end
return 1  -- allowed
```


## Resources

- [Redis Scripting with Lua](https://redis.io/docs/latest/develop/interact/programmability/eval-intro/) — Redis Lua scripting patterns and reference

---

> 📘 *This lesson is part of the [Scripting, Transactions, and Programmability](https://stanza.dev/courses/redis-scripting) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*