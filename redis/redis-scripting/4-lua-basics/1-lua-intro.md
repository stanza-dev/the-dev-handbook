---
source_course: "redis-scripting"
source_lesson: "redis-scripting-lua-intro"
---

# Introduction to Lua Scripts

Lua scripting allows you to run complex logic atomically on the Redis server, with access to multiple keys and custom computations.

## Why Lua Scripts?

1. **True Atomicity**: Entire script runs without interruption
2. **Reduced Network**: One round-trip for complex operations
3. **Server-Side Logic**: Compute on data without transferring it
4. **Conditional Operations**: Read, decide, write atomically

## EVAL Command

```redis
EVAL "return 'Hello, Lua!'" 0
# Returns: "Hello, Lua!"

# Syntax:
# EVAL script numkeys [key ...] [arg ...]
```

## Accessing Keys and Arguments

```redis
# KEYS array: keys passed to script
# ARGV array: additional arguments

EVAL "return {KEYS[1], KEYS[2], ARGV[1], ARGV[2]}" 2 key1 key2 arg1 arg2
# Returns: ["key1", "key2", "arg1", "arg2"]
```

## Calling Redis Commands

Use `redis.call()` to execute Redis commands:

```lua
-- Set a key and return its value
redis.call('SET', KEYS[1], ARGV[1])
return redis.call('GET', KEYS[1])
```

```redis
EVAL "redis.call('SET', KEYS[1], ARGV[1]); return redis.call('GET', KEYS[1])" 1 mykey myvalue
# Returns: "myvalue"
```

## redis.call vs redis.pcall

```lua
-- redis.call: Raises error on failure (stops script)
redis.call('SET', KEYS[1], ARGV[1])

-- redis.pcall: Returns error object (script continues)
local result = redis.pcall('INCR', 'non_numeric_key')
if result.err then
    return "Error: " .. result.err
end
```

## Lua Basics for Redis

### Variables and Types

```lua
local count = 10           -- number
local name = "Alice"       -- string
local flag = true          -- boolean
local items = {1, 2, 3}    -- table (array)
local user = {             -- table (object)
    name = "Alice",
    age = 30
}
```

### Conditionals

```lua
local balance = tonumber(redis.call('GET', KEYS[1]))

if balance == nil then
    return "Key not found"
elseif balance < 0 then
    return "Negative balance"
else
    return balance
end
```

### Loops

```lua
-- For loop
for i = 1, #KEYS do
    redis.call('DEL', KEYS[i])
end

-- While loop
local cursor = "0"
repeat
    local result = redis.call('SCAN', cursor, 'MATCH', 'temp:*')
    cursor = result[1]
    local keys = result[2]
    for i, key in ipairs(keys) do
        redis.call('DEL', key)
    end
until cursor == "0"
```

## Practical Example: Rate Limiter

```lua
-- rate_limit.lua
-- KEYS[1] = rate limit key
-- ARGV[1] = limit
-- ARGV[2] = window (seconds)

local current = redis.call('GET', KEYS[1])

if current == false then
    -- First request in window
    redis.call('SET', KEYS[1], 1, 'EX', ARGV[2])
    return 1
elseif tonumber(current) < tonumber(ARGV[1]) then
    -- Under limit
    return redis.call('INCR', KEYS[1])
else
    -- Over limit
    return -1
end
```

```redis
EVAL "<script>" 1 rate:user:1001 100 60
# Returns count if allowed, -1 if rate limited
```

## Script Caching with EVALSHA

```redis
# Load script and get SHA1
SCRIPT LOAD "return redis.call('GET', KEYS[1])"
# Returns: "a42059b356c875f0717db19a51f6aaa9161e77a2"

# Execute using SHA1 (faster, less bandwidth)
EVALSHA a42059b356c875f0717db19a51f6aaa9161e77a2 1 mykey
```

📖 [Redis Scripting](https://redis.io/docs/latest/develop/interact/programmability/eval-intro/)

## Resources

- [Redis Scripting](https://redis.io/docs/latest/develop/interact/programmability/eval-intro/) — Introduction to Redis Lua scripting

---

> 📘 *This lesson is part of the [Scripting, Transactions, and Programmability](https://stanza.dev/courses/redis-scripting) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*