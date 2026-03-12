---
source_course: "redis-scripting"
source_lesson: "redis-scripting-lua-intro"
---

# Introduction to Lua Scripts

## Introduction

Lua scripting allows you to run complex logic atomically on the Redis server. Unlike MULTI/EXEC which just queues commands, a Lua script can read data, branch on conditions, loop, and write — all in one uninterruptible operation. This is the most powerful form of atomicity Redis offers.

## Key Concepts

- **EVAL** — execute a Lua script inline: `EVAL script numkeys [key...] [arg...]`
- **KEYS array** — Lua 1-indexed array of Redis keys passed to the script; access via `KEYS[1]`, `KEYS[2]`
- **ARGV array** — 1-indexed array of additional arguments; access via `ARGV[1]`, `ARGV[2]`
- **redis.call()** — calls a Redis command from Lua; raises a Lua error if the command fails
- **Atomic execution** — no other Redis commands run while the script executes

## Real World Context

You need to implement a "claim once" feature: the first user to claim a reward gets it; everyone else is rejected. This requires atomically checking and setting a flag. A single Lua script does this perfectly — no WATCH retry loop needed.

## Deep Dive

The EVAL syntax:

```bash
EVAL "return redis.call('SET', KEYS[1], ARGV[1])" 1 mykey myvalue
#     ^script                                       ^ numkeys ^ key ^ arg
```

`numkeys` tells Redis how many of the following tokens are keys (KEYS) vs. arguments (ARGV). This matters for Redis Cluster, which must route the command to the correct node.

A more realistic example — claim once:

```lua
-- KEYS[1] = reward key, ARGV[1] = user_id
local already_claimed = redis.call('GET', KEYS[1])
if already_claimed then
    return 0  -- already claimed
end
redis.call('SET', KEYS[1], ARGV[1])
return 1  -- claimed successfully
```

Execute from the CLI:

```bash
EVAL "local c = redis.call('GET', KEYS[1]); if c then return 0 end; redis.call('SET', KEYS[1], ARGV[1]); return 1" 1 reward:summer-promo user:42
```

For multi-line scripts, use a file:

```bash
redis-cli --eval claim_once.lua reward:summer-promo , user:42
# Note: comma separates KEYS from ARGV in redis-cli --eval syntax
```

## Common Pitfalls

1. **Forgetting numkeys.** If you pass `EVAL script 0 myarg`, `myarg` is an ARGV (not a KEYS entry). Getting numkeys wrong causes cluster routing errors.
2. **Using global variables.** Lua scripts in Redis must be stateless between calls. Global variable state does not persist across EVAL calls.
3. **Accessing KEYS[0] instead of KEYS[1].** Lua uses 1-based indexing. `KEYS[0]` is nil, not the first key.

## Best Practices

1. Always declare all accessed Redis keys in the KEYS array, even when not strictly required (for cluster compatibility).
2. Keep scripts short and focused — long scripts block Redis for all other clients.
3. Cache scripts with SCRIPT LOAD and use EVALSHA in production to avoid sending the full script text on every call.

## Summary

- EVAL executes Lua scripts atomically on the Redis server
- KEYS[1..n] are the Redis keys; ARGV[1..m] are the arguments (both 1-indexed)
- numkeys separates KEYS from ARGV in the EVAL command
- redis.call() raises on error; redis.pcall() returns an error table
- Scripts block all other commands while running — keep them fast

## Code Examples

**Basic EVAL usage and claim-once example**

```bash
# Basic EVAL syntax
EVAL "return redis.call('SET', KEYS[1], ARGV[1])" 1 mykey myvalue

# Claim-once script
EVAL "local c = redis.call('GET', KEYS[1]); if c then return 0 end; redis.call('SET', KEYS[1], ARGV[1]); return 1" 1 reward:summer-promo user:42
```

**Multi-line claim-once Lua script**

```lua
-- claim_once.lua
-- KEYS[1] = reward key, ARGV[1] = user_id
local already_claimed = redis.call('GET', KEYS[1])
if already_claimed then
    return 0
end
redis.call('SET', KEYS[1], ARGV[1])
return 1
```


## Resources

- [Redis Scripting with Lua](https://redis.io/docs/latest/develop/interact/programmability/eval-intro/) — Introduction to Redis Lua scripting

---

> 📘 *This lesson is part of the [Scripting, Transactions, and Programmability](https://stanza.dev/courses/redis-scripting) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*