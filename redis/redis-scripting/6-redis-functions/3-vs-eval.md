---
source_course: "redis-scripting"
source_lesson: "redis-scripting-functions-vs-eval"
---

# Functions vs EVAL: When to Use Each

## Introduction

Redis provides two ways to run Lua logic on the server: EVAL (inline scripts) and Redis Functions. Both execute atomically, but they have fundamentally different persistence, management, and operational characteristics. Knowing when to use each prevents operational headaches in production.

## Key Concepts

- **EVAL** — sends script text on every call (or uses EVALSHA after SCRIPT LOAD); not persisted; cache cleared on restart
- **Redis Functions** — stored in named libraries in the database; persisted in RDB; replicated to replicas; managed independently of application deployments
- **One-off scripts** — temporary or exploratory logic; use EVAL
- **Production business logic** — reusable, versioned, persistent; use Functions

## Real World Context

A developer debugging a production issue needs to run a one-time script to inspect and repair some data. EVAL is perfect — fire and forget. But a rate limiter that runs on every API request should be a Function: it persists through Redis restarts, is available on all replicas, and can be updated independently.

## Deep Dive

Persistence comparison:

```bash
# EVAL / EVALSHA lifecycle:
SCRIPT LOAD "return 'hello'"       # Cache in memory
EVALSHA <sha1> 0                   # Works until Redis restarts
# After restart: NOSCRIPT error — must SCRIPT LOAD again

# Function lifecycle:
FUNCTION LOAD "#!lua name=greet\nredis.register_function('hello', function(k,a) return 'hello' end)"
FCALL hello 0                      # Works now
# After restart: still works — saved in RDB
# After replica failover: still works — replicated
```

Replication:
- **EVAL/EVALSHA**: The script cache is NOT replicated. Each replica needs its own SCRIPT LOAD.
- **Functions**: The library is part of the dataset — loaded once on primary, automatically replicated.

Deployment workflow comparison:

| Aspect | EVAL | Functions |
|---|---|---|
| Script sent on each call | Yes (EVAL) / No (EVALSHA) | No — called by name |
| Survives restart | No | Yes (in RDB) |
| Replicated | No | Yes |
| Updatable without app redeploy | No | Yes (FUNCTION LOAD REPLACE) |
| Good for | One-off, debugging | Production business logic |

## Common Pitfalls

1. **Using EVAL for production high-frequency code.** Every EVAL call sends the full script; use EVALSHA (or Functions) in production.
2. **Using Functions for temporary debugging scripts.** Functions persist and replicate — loading a debug script as a Function clutters the library namespace and replicates unnecessarily.
3. **Forgetting that FUNCTION FLUSH on primary replicates to replicas.** Flushing functions in production removes them everywhere.

## Best Practices

1. Use EVAL for debugging, one-off data migrations, and experimentation.
2. Use Functions for any logic called frequently in production: rate limiters, counters, business rules.
3. Deploy function libraries through your CI/CD pipeline, not manually — treat them as code artifacts.

## Summary

- EVAL is for one-off and exploratory use; script cache is not persisted or replicated
- Functions are for production reusable logic; persisted in RDB and replicated automatically
- EVALSHA reduces bandwidth compared to EVAL but still requires reload after restart
- Functions can be updated without application redeployment via FUNCTION LOAD REPLACE
- Treat function libraries as versioned code artifacts, deployed through CI/CD

## Code Examples

**EVAL vs EVALSHA vs FCALL comparison**

```bash
# EVAL: one-off use (not persisted)
EVAL "return redis.call('KEYS', '*')" 0

# EVALSHA: reduces bandwidth but lost on restart
SCRIPT LOAD "return redis.call('GET', KEYS[1])"
# => sha1hash
EVALSHA sha1hash 1 mykey

# Function: persisted and replicated
FUNCTION LOAD "#!lua name=utils\nredis.register_function('safe_get', function(keys, args) local v = redis.call('GET', keys[1]); if v == false then return args[1] end; return v end)"
FCALL safe_get 1 mykey default_value
```


## Resources

- [Redis Functions](https://redis.io/docs/latest/develop/interact/programmability/functions-intro/) — Redis Functions vs EVAL comparison and guide
- [Redis Scripting with EVAL](https://redis.io/docs/latest/develop/interact/programmability/eval-intro/) — EVAL scripting reference

---

> 📘 *This lesson is part of the [Scripting, Transactions, and Programmability](https://stanza.dev/courses/redis-scripting) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*