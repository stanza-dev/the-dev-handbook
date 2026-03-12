---
source_course: "redis-scripting"
source_lesson: "redis-scripting-lua-keys-argv"
---

# KEYS and ARGV in Lua Scripts

## Introduction

Every Redis Lua script receives two special 1-indexed arrays: KEYS and ARGV. Understanding how they work — and why all Redis keys must be declared in KEYS — is essential for writing correct, cluster-compatible scripts.

## Key Concepts

- **KEYS array** — Redis keys the script will access; declared so Redis can route to the correct cluster node
- **ARGV array** — additional arguments (user IDs, thresholds, values) that are not keys
- **1-indexed** — Lua arrays start at index 1, not 0; `KEYS[1]` is the first key, `KEYS[0]` is nil
- **numkeys** — the integer in the EVAL command that tells Redis how many tokens are KEYS vs. ARGV
- **Cluster compatibility** — Redis Cluster requires all accessed keys to be in the same slot; declaring them in KEYS enables this check

## Real World Context

You are writing a script that reads a user's balance (one key) and a product's price (another key) and conditionally decrements the balance. Both keys must be in KEYS — if you hardcode a key name inside the script instead, Redis Cluster cannot route the command correctly.

## Deep Dive

The EVAL command format:

```bash
EVAL script numkeys key1 key2 ... arg1 arg2 ...
```

Inside the script, these are available as:
- `KEYS[1]` = key1, `KEYS[2]` = key2, ...
- `ARGV[1]` = arg1, `ARGV[2]` = arg2, ...

Example — purchase script with two keys:

```bash
EVAL "
local balance = tonumber(redis.call('GET', KEYS[1])) or 0
local price = tonumber(redis.call('GET', KEYS[2])) or 0
if balance < price then
    return -1
end
redis.call('DECRBY', KEYS[1], price)
return balance - price
" 2 user:42:balance product:99:price
```

Here `numkeys` is 2, so `KEYS[1]` = `user:42:balance` and `KEYS[2]` = `product:99:price`. There are no ARGV entries.

With both KEYS and ARGV:

```bash
EVAL "
local current = tonumber(redis.call('GET', KEYS[1])) or 0
local threshold = tonumber(ARGV[1])
if current >= threshold then
    redis.call('SET', KEYS[2], ARGV[2])
end
return current
" 2 counter:visits alert:status 100 triggered
```

Here `numkeys` is 2: `KEYS[1]`=counter:visits, `KEYS[2]`=alert:status, `ARGV[1]`=100, `ARGV[2]`=triggered.

## Common Pitfalls

1. **Hardcoding key names inside the script.** `redis.call('GET', 'user:42:balance')` bypasses the KEYS array. Redis Cluster will refuse to execute such scripts because it cannot verify key locality.
2. **Off-by-one errors.** `KEYS[0]` and `ARGV[0]` are both nil. Always start from index 1.
3. **Miscounting numkeys.** If numkeys is wrong, some KEYS entries become ARGV or vice versa, causing subtle bugs.

## Best Practices

1. Always pass all accessed Redis key names through KEYS, even in non-clustered deployments — it makes scripts future-proof for cluster migration.
2. Use ARGV for non-key parameters: thresholds, user IDs, flag values, etc.
3. Add a comment at the top of each script listing what KEYS[1..n] and ARGV[1..m] represent.

## Summary

- KEYS[1..n] are Redis key names; ARGV[1..m] are non-key arguments
- Lua arrays are 1-indexed — KEYS[0] and ARGV[0] are always nil
- numkeys in EVAL tells Redis the boundary between KEYS and ARGV tokens
- All accessed key names must be in KEYS for Redis Cluster compatibility
- Hardcoding key names inside a script breaks cluster routing

## Code Examples

**Script with two KEYS and no ARGV**

```bash
# numkeys=2: KEYS[1]=user:42:balance, KEYS[2]=product:99:price
EVAL "
local balance = tonumber(redis.call('GET', KEYS[1])) or 0
local price = tonumber(redis.call('GET', KEYS[2])) or 0
if balance < price then return -1 end
redis.call('DECRBY', KEYS[1], price)
return balance - price
" 2 user:42:balance product:99:price
```

**Script using both KEYS and ARGV arrays**

```lua
-- KEYS[1] = counter key, KEYS[2] = alert key
-- ARGV[1] = threshold, ARGV[2] = alert value
local current = tonumber(redis.call('GET', KEYS[1])) or 0
local threshold = tonumber(ARGV[1])
if current >= threshold then
    redis.call('SET', KEYS[2], ARGV[2])
end
return current
```


## Resources

- [Redis Lua API](https://redis.io/docs/latest/develop/interact/programmability/lua-api/) — Redis Lua API reference including KEYS and ARGV

---

> 📘 *This lesson is part of the [Scripting, Transactions, and Programmability](https://stanza.dev/courses/redis-scripting) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*