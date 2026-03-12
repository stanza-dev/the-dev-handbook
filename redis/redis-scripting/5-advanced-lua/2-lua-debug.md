---
source_course: "redis-scripting"
source_lesson: "redis-scripting-lua-debug"
---

# Script Flags and Debugging

## Introduction

Redis 7.0 introduced script flags — special directives in a Lua script header that declare the script's access patterns to Redis. These flags enable safer execution on replicas and improve cluster compatibility. This lesson also covers the essential SCRIPT management commands for production.

## Key Concepts

- **Script flags** — header line `#!lua flags=...` declaring script behavior
- **no-writes** — script makes no writes; safe for EVAL_RO and replica execution
- **allow-oom** — script can run even when Redis is over its maxmemory limit
- **SCRIPT LOAD** — caches a script and returns its SHA1 hash for use with EVALSHA
- **SCRIPT EXISTS** — checks if given SHA1 hashes are in the server's script cache
- **SCRIPT FLUSH** — clears the entire script cache

## Real World Context

Your application has a read-heavy Lua script that fetches aggregated analytics from multiple keys. Without script flags, Redis cannot confirm the script is safe for replica execution. Adding `#!lua flags=no-writes` allows replicas to serve this script, reducing load on the primary.

## Deep Dive

Script flags are placed on the very first line:

```lua
#!lua flags=no-writes
-- This script only reads; safe for replica execution
-- KEYS[1] = the counter key
local val = redis.call('GET', KEYS[1])
return val or '0'
```

Available flags:

- `no-writes` — script makes no write calls; allows EVAL_RO
- `allow-oom` — script can execute when server is at maxmemory
- `allow-repl` — enable manual replication control (advanced)
- `no-cluster` — disallow execution in cluster mode

Managing the script cache:

```bash
# Load and cache a script
SCRIPT LOAD "#!lua flags=no-writes\nreturn redis.call('GET', KEYS[1])"
# => "abc123sha1"

# Verify it's cached
SCRIPT EXISTS abc123sha1
# => 1) (integer) 1

# Execute by SHA1 (no script text sent over network)
EVALSHA abc123sha1 1 mykey

# Clear all cached scripts (use with caution)
SCRIPT FLUSH
```

## Common Pitfalls

1. **Placing flags on any line other than the first.** The `#!lua flags=...` directive must be the very first line; otherwise Redis ignores it completely.
2. **Declaring no-writes but calling a write command.** If the script calls SET with no-writes declared, Redis raises an error at runtime on replicas.
3. **Running SCRIPT FLUSH without a reload plan.** After SCRIPT FLUSH, all EVALSHA calls return NOSCRIPT until scripts are re-uploaded with SCRIPT LOAD.

## Best Practices

1. Add `no-writes` to all read-only scripts to unlock replica-safe execution and improve horizontal scaling.
2. Implement NOSCRIPT retry logic in client code: catch NOSCRIPT, call SCRIPT LOAD, then retry EVALSHA.
3. Use SCRIPT EXISTS in health checks to verify scripts are loaded before traffic starts.

## Summary

- Script flags (`#!lua flags=...`) must be on the first line of the script
- `no-writes` enables EVAL_RO and safe replica execution
- SCRIPT LOAD caches a script server-side and returns its SHA1
- SCRIPT EXISTS checks whether scripts are in cache (1 = cached, 0 = not)
- SCRIPT FLUSH clears all cached scripts; plan a reload procedure before using it

## Code Examples

**Lua script with no-writes flag for replica safety**

```lua
#!lua flags=no-writes
-- Read-only: safe for replica execution
-- KEYS[1] = the key to fetch
local val = redis.call('GET', KEYS[1])
return val or '0'
```

**Script cache management: LOAD, EXISTS, EVALSHA**

```bash
# Load and cache
SCRIPT LOAD "#!lua flags=no-writes\nreturn redis.call('GET', KEYS[1])"
# => 'abc123sha1'

# Verify cached
SCRIPT EXISTS abc123sha1
# => 1) (integer) 1

# Run by SHA1
EVALSHA abc123sha1 1 mykey
```


## Resources

- [Lua API Reference](https://redis.io/docs/latest/develop/interact/programmability/lua-api/) — Redis Lua API including script flags
- [Redis Scripting Commands](https://redis.io/docs/latest/commands/?group=scripting) — SCRIPT subcommands reference

---

> 📘 *This lesson is part of the [Scripting, Transactions, and Programmability](https://stanza.dev/courses/redis-scripting) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*