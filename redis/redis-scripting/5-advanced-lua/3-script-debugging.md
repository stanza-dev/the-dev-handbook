---
source_course: "redis-scripting"
source_lesson: "redis-scripting-script-debugging"
---

# Debugging Scripts with LDB and SCRIPT KILL

## Introduction

When Lua scripts misbehave in production, Redis provides tools to diagnose and recover: the Lua Debugger (LDB) for step-through debugging in development, SCRIPT KILL for terminating stuck scripts, and the lua-time-limit configuration that governs how long scripts can run before Redis stops accepting new commands.

## Key Concepts

- **lua-time-limit** — config option (default 5000ms) after which a stuck script causes Redis to stop accepting most commands
- **SCRIPT KILL** — terminates a running script that has not yet performed any write operations
- **Redis Lua Debugger (LDB)** — interactive step-through debugger invoked with `redis-cli --ldb --eval`
- **NOSCRIPT error** — returned when EVALSHA is called with a SHA1 not in the cache (e.g., after restart or SCRIPT FLUSH)

## Real World Context

A Lua script deployed overnight starts hanging in production due to an edge-case infinite loop. Other clients can no longer execute commands. You use SCRIPT KILL to unblock Redis, then reproduce the bug locally with the LDB to find and fix the loop.

## Deep Dive

Script timeout handling:

```bash
# Check current timeout setting
CONFIG GET lua-time-limit
# => 1) "lua-time-limit" 2) "5000"

# Once lua-time-limit is exceeded, Redis accepts ONLY:
#   SCRIPT KILL   (if script has not written data)
#   SHUTDOWN NOSAVE  (emergency shutdown, data lost)

# Kill a stuck read-only script
SCRIPT KILL
# => OK

# If script already wrote data:
# (error) UNKILLABLE Not possible to kill a write script
# At that point, only SHUTDOWN NOSAVE stops it
```

Using the Lua Debugger (never use on production):

```bash
# Save script to a file
cat > myscript.lua << 'EOF'
local val = redis.call('GET', KEYS[1])
local count = tonumber(val) or 0
redis.call('SET', KEYS[2], count * 2)
return count
EOF

# Start interactive debug session against a LOCAL Redis
redis-cli --ldb --eval myscript.lua key1 key2 , arg1
```

LDB commands reference:

| Command | Action |
|---------|--------|
| s | Step: execute one Lua line |
| n | Next: step over function calls |
| c | Continue: run to next breakpoint or end |
| p expr | Print: evaluate and print a Lua expression |
| b N | Breakpoint: set at line N |
| del N | Delete breakpoint at line N |
| q | Quit: abort the script |

Handling NOSCRIPT in application code:

```python
import redis

def safe_evalsha(r, sha, script, numkeys, *args):
    try:
        return r.evalsha(sha, numkeys, *args)
    except redis.ResponseError as e:
        if 'NOSCRIPT' in str(e):
            new_sha = r.script_load(script)
            return r.evalsha(new_sha, numkeys, *args)
        raise
```

## Common Pitfalls

1. **Debugging against a production Redis.** LDB blocks the entire server. Always debug locally or in a dedicated staging environment.
2. **Setting lua-time-limit too high.** A very high timeout means stuck scripts block Redis for longer before SCRIPT KILL becomes available.
3. **Not handling NOSCRIPT in production.** After any Redis restart, script caches are cleared. Every EVALSHA call will fail until scripts are reloaded.

## Best Practices

1. Always write test scripts that cover edge cases (nil keys, empty results, type mismatches) before deploying.
2. Implement NOSCRIPT retry logic for all production EVALSHA calls.
3. Use the LDB exclusively in local development; invest time here to avoid production incidents.

## Summary

- lua-time-limit (default 5s) is when Redis accepts SCRIPT KILL for stuck scripts
- SCRIPT KILL only terminates read-only scripts; write scripts require SHUTDOWN NOSAVE
- The LDB provides interactive step-through debugging — use it in development only
- LDB blocks the Redis server; never run it against production
- Handle NOSCRIPT errors by catching the error, reloading the script, and retrying EVALSHA

## Code Examples

**Script timeout management and LDB usage**

```bash
# Check lua-time-limit
CONFIG GET lua-time-limit
# => 5000 ms

# Kill a stuck read-only script
SCRIPT KILL

# Interactive debugger (local/staging only!)
redis-cli --ldb --eval myscript.lua key1 key2 , arg1
# LDB commands: s=step, n=next, c=continue, p expr=print, b N=breakpoint, q=quit
```


## Resources

- [Redis Scripting Commands](https://redis.io/docs/latest/commands/?group=scripting) — SCRIPT KILL and debugging commands reference

---

> 📘 *This lesson is part of the [Scripting, Transactions, and Programmability](https://stanza.dev/courses/redis-scripting) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*