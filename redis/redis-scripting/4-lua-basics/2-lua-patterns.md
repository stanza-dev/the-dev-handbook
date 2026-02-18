---
source_course: "redis-scripting"
source_lesson: "redis-scripting-lua-patterns"
---

# Lua Script Patterns

Common patterns and best practices for Lua scripts in Redis.

## Pattern 1: Conditional Update

```lua
-- Only update if current value matches expected
-- KEYS[1] = key
-- ARGV[1] = expected value
-- ARGV[2] = new value

local current = redis.call('GET', KEYS[1])
if current == ARGV[1] then
    redis.call('SET', KEYS[1], ARGV[2])
    return 1
else
    return 0
end
```

## Pattern 2: Get-Set-If-Not-Exists

```lua
-- Get value, set default if not exists
-- KEYS[1] = key
-- ARGV[1] = default value
-- ARGV[2] = TTL (optional)

local value = redis.call('GET', KEYS[1])
if value == false then
    if ARGV[2] then
        redis.call('SET', KEYS[1], ARGV[1], 'EX', ARGV[2])
    else
        redis.call('SET', KEYS[1], ARGV[1])
    end
    return ARGV[1]
else
    return value
end
```

## Pattern 3: Atomic Increment with Limit

```lua
-- Increment but don't exceed limit
-- KEYS[1] = counter key
-- ARGV[1] = increment amount
-- ARGV[2] = maximum value

local current = tonumber(redis.call('GET', KEYS[1])) or 0
local increment = tonumber(ARGV[1])
local maximum = tonumber(ARGV[2])

local new_value = current + increment
if new_value > maximum then
    new_value = maximum
end

redis.call('SET', KEYS[1], new_value)
return new_value
```

## Pattern 4: Sliding Window Rate Limiter

```lua
-- KEYS[1] = rate limit key (sorted set)
-- ARGV[1] = current timestamp (ms)
-- ARGV[2] = window size (ms)
-- ARGV[3] = max requests

local now = tonumber(ARGV[1])
local window = tonumber(ARGV[2])
local limit = tonumber(ARGV[3])
local clearBefore = now - window

-- Remove old entries
redis.call('ZREMRANGEBYSCORE', KEYS[1], 0, clearBefore)

-- Count requests in window
local count = redis.call('ZCARD', KEYS[1])

if count < limit then
    -- Add this request
    redis.call('ZADD', KEYS[1], now, now .. '-' .. math.random())
    redis.call('EXPIRE', KEYS[1], math.ceil(window/1000))
    return 1  -- Allowed
else
    return 0  -- Rate limited
end
```

## Pattern 5: Distributed Lock

```lua
-- Acquire lock
-- KEYS[1] = lock key
-- ARGV[1] = lock token (unique identifier)
-- ARGV[2] = TTL in seconds

local current = redis.call('GET', KEYS[1])
if current == false then
    redis.call('SET', KEYS[1], ARGV[1], 'EX', ARGV[2])
    return 1  -- Lock acquired
else
    return 0  -- Lock held by someone else
end
```

```lua
-- Release lock (only if we own it)
-- KEYS[1] = lock key
-- ARGV[1] = our lock token

local current = redis.call('GET', KEYS[1])
if current == ARGV[1] then
    redis.call('DEL', KEYS[1])
    return 1  -- Lock released
else
    return 0  -- Lock not ours
end
```

## Pattern 6: Batch Operations

```lua
-- Delete all keys matching pattern
-- ARGV[1] = pattern
-- ARGV[2] = count per iteration

local cursor = "0"
local deleted = 0

repeat
    local result = redis.call('SCAN', cursor, 'MATCH', ARGV[1], 'COUNT', ARGV[2])
    cursor = result[1]
    local keys = result[2]
    
    if #keys > 0 then
        deleted = deleted + redis.call('DEL', unpack(keys))
    end
until cursor == "0"

return deleted
```

## Best Practices

1. **Keep scripts short**: Long scripts block Redis
2. **Use KEYS array for keys**: Required for Redis Cluster
3. **Handle nil values**: `GET` returns `false` in Lua for non-existent keys
4. **Type conversion**: Use `tonumber()` for numeric operations
5. **Test thoroughly**: Debug with `redis.log(redis.LOG_WARNING, "message")`

📖 [Lua Script Examples](https://redis.io/docs/latest/develop/interact/programmability/lua-api/)

---

> 📘 *This lesson is part of the [Scripting, Transactions, and Programmability](https://stanza.dev/courses/redis-scripting) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*