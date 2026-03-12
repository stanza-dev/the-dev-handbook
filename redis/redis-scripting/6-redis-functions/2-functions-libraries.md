---
source_course: "redis-scripting"
source_lesson: "redis-scripting-functions-libraries"
---

# Managing Function Libraries

## Introduction

Redis Functions are organized into libraries — named collections that can hold multiple related functions. Managing the lifecycle of libraries (loading, updating, listing, deleting) is a core operational skill for teams using server-side logic in Redis.

## Key Concepts

- **FUNCTION LOAD** — loads a new library; fails if library already exists (use REPLACE to update)
- **FUNCTION LOAD REPLACE** — atomically replaces an existing library with a new version
- **FUNCTION LIST** — lists all libraries with their functions, engines, and metadata
- **FUNCTION DELETE libname** — removes a library and all its functions
- **FUNCTION FLUSH** — removes all libraries from all engines (destructive!)
- **FUNCTION STATS** — shows currently executing functions and engine stats

## Real World Context

Your team has deployed a library called `payments` to production. A bug fix is ready. You use `FUNCTION LOAD REPLACE` to atomically swap the old library with the fixed version — no downtime, no gap where the function is unavailable.

## Deep Dive

Loading a library with multiple functions:

```bash
FUNCTION LOAD "#!lua name=payments\n
redis.register_function('debit', function(keys, args)
    local amount = tonumber(args[1])
    local balance = tonumber(redis.call('GET', keys[1])) or 0
    if balance < amount then return -1 end
    return redis.call('DECRBY', keys[1], amount)
end)

redis.register_function('credit', function(keys, args)
    return redis.call('INCRBY', keys[1], tonumber(args[1]))
end)"
```

Updating the library in production:

```bash
FUNCTION LOAD REPLACE "#!lua name=payments\n
-- Updated version with logging
redis.register_function('debit', function(keys, args)
    local amount = tonumber(args[1])
    local balance = tonumber(redis.call('GET', keys[1])) or 0
    if balance < amount then
        redis.call('INCR', 'metrics:debit:rejected')
        return -1
    end
    redis.call('INCR', 'metrics:debit:approved')
    return redis.call('DECRBY', keys[1], amount)
end)

redis.register_function('credit', function(keys, args)
    return redis.call('INCRBY', keys[1], tonumber(args[1]))
end)"
```

Management commands:

```bash
# List all libraries and their functions
FUNCTION LIST

# List functions in a specific library
FUNCTION LIST LIBRARYNAME payments

# Show current stats
FUNCTION STATS

# Delete a specific library
FUNCTION DELETE payments

# Delete all libraries (use with extreme caution!)
FUNCTION FLUSH
```

## Common Pitfalls

1. **Using FUNCTION LOAD without REPLACE on an existing library.** Redis returns an error: `Library already exists`. Always use REPLACE when updating.
2. **Running FUNCTION FLUSH in production.** This removes all libraries instantly. Plan a reload procedure before running it.
3. **Not testing REPLACE in staging first.** A library with a syntax error in FUNCTION LOAD REPLACE will fail and the old library remains, but a runtime bug will go live immediately.

## Best Practices

1. Use FUNCTION LOAD REPLACE in deployment pipelines to atomically update libraries.
2. Include a version comment inside the library code for traceability.
3. Monitor FUNCTION STATS in production to detect long-running function executions.

## Summary

- Libraries are named; FUNCTION LOAD fails if the library already exists
- FUNCTION LOAD REPLACE atomically updates an existing library
- FUNCTION LIST shows all libraries and their registered functions
- FUNCTION DELETE removes one library; FUNCTION FLUSH removes all (destructive)
- FUNCTION STATS shows running function metadata and engine statistics

## Code Examples

**Library lifecycle management commands**

```bash
# Load a library (fails if already exists)
FUNCTION LOAD "#!lua name=payments\nredis.register_function('debit', function(keys, args) return 'ok' end)"

# Update existing library atomically
FUNCTION LOAD REPLACE "#!lua name=payments\nredis.register_function('debit', function(keys, args) return 'ok' end)"

# List all libraries
FUNCTION LIST

# Delete a library
FUNCTION DELETE payments

# Show stats
FUNCTION STATS
```


## Resources

- [Redis Functions](https://redis.io/docs/latest/develop/interact/programmability/functions-intro/) — Redis Functions introduction and management
- [FUNCTION Commands](https://redis.io/commands/?group=scripting) — Full reference for FUNCTION subcommands

---

> 📘 *This lesson is part of the [Scripting, Transactions, and Programmability](https://stanza.dev/courses/redis-scripting) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*