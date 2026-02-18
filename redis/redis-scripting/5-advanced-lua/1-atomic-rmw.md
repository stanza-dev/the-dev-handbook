---
source_course: "redis-scripting"
source_lesson: "redis-scripting-atomic-rmw"
---

# Atomic Read-Modify-Write

Lua scripts excel at operations that read data, make decisions, and write atomically.

## Conditional Update Pattern

```lua
-- update_if_greater.lua
-- Only update if new value is greater than current
-- KEYS[1] = key to update
-- ARGV[1] = new value

local current = redis.call('GET', KEYS[1])
local new_val = tonumber(ARGV[1])

if current == false then
    -- Key doesn't exist, set it
    redis.call('SET', KEYS[1], new_val)
    return new_val
end

local current_val = tonumber(current)
if new_val > current_val then
    redis.call('SET', KEYS[1], new_val)
    return new_val
else
    return current_val
end
```

## Complex Conditional Logic

```lua
-- purchase.lua
-- Atomic purchase: check balance, check stock, update both
-- KEYS[1] = user balance key
-- KEYS[2] = product stock key
-- ARGV[1] = price
-- ARGV[2] = quantity

local balance = tonumber(redis.call('GET', KEYS[1])) or 0
local stock = tonumber(redis.call('GET', KEYS[2])) or 0
local price = tonumber(ARGV[1])
local quantity = tonumber(ARGV[2])

local total_cost = price * quantity

-- Check both conditions
if balance < total_cost then
    return redis.error_reply('INSUFFICIENT_BALANCE')
end

if stock < quantity then
    return redis.error_reply('OUT_OF_STOCK')
end

-- Both conditions met, execute atomically
redis.call('DECRBY', KEYS[1], total_cost)
redis.call('DECRBY', KEYS[2], quantity)

return cjson.encode({
    success = true,
    new_balance = balance - total_cost,
    new_stock = stock - quantity
})
```

## Sliding Window Rate Limiter

```lua
-- sliding_window.lua
-- Precise rate limiting using sorted sets
-- KEYS[1] = rate limit key
-- ARGV[1] = window size (seconds)
-- ARGV[2] = max requests
-- ARGV[3] = current timestamp
-- ARGV[4] = unique request ID

local key = KEYS[1]
local window = tonumber(ARGV[1])
local limit = tonumber(ARGV[2])
local now = tonumber(ARGV[3])
local request_id = ARGV[4]

local window_start = now - window

-- Remove old entries
redis.call('ZREMRANGEBYSCORE', key, '-inf', window_start)

-- Count current requests
local current = redis.call('ZCARD', key)

if current >= limit then
    return 0  -- Rate limited
end

-- Add this request
redis.call('ZADD', key, now, request_id)
redis.call('EXPIRE', key, window)

return limit - current  -- Remaining requests
```

## JSON Processing with cjson

```lua
-- process_json.lua
-- Parse, modify, and store JSON
-- KEYS[1] = JSON document key
-- ARGV[1] = path to update
-- ARGV[2] = new value

local json_str = redis.call('GET', KEYS[1])
if not json_str then
    return redis.error_reply('KEY_NOT_FOUND')
end

local data = cjson.decode(json_str)

-- Navigate path and update
local path = ARGV[1]
local value = ARGV[2]

-- Simple single-level path update
data[path] = value

local new_json = cjson.encode(data)
redis.call('SET', KEYS[1], new_json)

return new_json
```

📖 [Lua Scripting](https://redis.io/docs/latest/develop/interact/programmability/eval-intro/)

## Resources

- [Redis Scripting](https://redis.io/docs/latest/develop/interact/programmability/eval-intro/) — Redis Lua scripting documentation

---

> 📘 *This lesson is part of the [Scripting, Transactions, and Programmability](https://stanza.dev/courses/redis-scripting) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*