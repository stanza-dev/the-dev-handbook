---
source_course: "redis-scripting"
source_lesson: "redis-scripting-functions-intro"
---

# Redis Functions Overview

## Introduction

Redis Functions (introduced in Redis 7.0) provide a modern, persistent alternative to EVAL scripts. Functions are stored inside named libraries in the database, persist across restarts, and are replicated to replicas — solving the biggest operational pain of EVAL scripts, which must be reloaded after every restart.

## Key Concepts

- **Library** — a named container holding one or more function definitions; loaded with FUNCTION LOAD
- **Function** — a named callable registered within a library; invoked with FCALL
- **Persistence** — functions are saved in RDB snapshots and replicated, unlike EVAL scripts which only live in memory
- **FCALL** — executes a function: `FCALL funcname numkeys [key...] [arg...]`
- **FCALL_RO** — read-only variant for replica execution

## Real World Context

A team deploys a suite of business logic scripts to Redis. With EVAL, every Redis restart requires the application to reload all scripts with SCRIPT LOAD before the first EVALSHA call. With Functions, the library is part of the database — after a restart, functions are immediately available with no application-side reload logic.

## Deep Dive

Loading a function library:

```bash
FUNCTION LOAD "#!lua name=mylib\n
redis.register_function('get_with_default', function(keys, args)
    local val = redis.call('GET', keys[1])
    if val == false then
        return args[1]
    end
    return val
end)"
```

The `#!lua name=mylib` shebang line declares the engine (lua) and the library name. Functions registered inside are accessible by name.

Calling the function:

```bash
# FCALL funcname numkeys key1 arg1
FCALL get_with_default 1 user:42:theme dark
# Returns: 'dark' if key doesn't exist, or the stored value if it does
```

Read-only call on a replica:

```bash
FCALL_RO get_with_default 1 user:42:theme dark
```

Checking loaded functions:

```bash
FUNCTION LIST
# Returns library names, engines, and function names
```

## Common Pitfalls

1. **Trying to FCALL a function before loading its library.** Functions must be loaded first. If the function doesn't exist, FCALL returns an error: `ERR Library not found`.
2. **Using FCALL_RO for a write function.** FCALL_RO enforces a no-writes policy. If the function calls SET or other write commands, Redis returns an error.
3. **Forgetting the shebang line.** The `#!lua name=libraryname` header is required in every FUNCTION LOAD payload — without it, Redis rejects the load.

## Best Practices

1. Organize related functions into a single named library to reduce management overhead.
2. Use FCALL_RO for read-only functions to allow execution on read replicas and improve horizontal scalability.
3. Version your library name (e.g., `mylib_v2`) to manage rolling updates without breaking existing callers.

## Summary

- Functions are stored in named libraries that persist across Redis restarts
- FUNCTION LOAD loads a library; the shebang `#!lua name=libname` is required
- FCALL calls a function; FCALL_RO enforces no writes
- Functions are replicated to replicas, unlike EVAL script caches
- FUNCTION LIST shows all loaded libraries and their functions

## Code Examples

**Loading and calling a Redis Function**

```bash
# Load a function library
FUNCTION LOAD "#!lua name=mylib\nredis.register_function('get_with_default', function(keys, args) local val = redis.call('GET', keys[1]); if val == false then return args[1] end; return val end)"

# Call the function
FCALL get_with_default 1 user:42:theme dark

# List all loaded functions
FUNCTION LIST
```


## Resources

- [Redis Functions](https://redis.io/docs/latest/develop/interact/programmability/functions-intro/) — Guide to Redis Functions

---

> 📘 *This lesson is part of the [Scripting, Transactions, and Programmability](https://stanza.dev/courses/redis-scripting) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*