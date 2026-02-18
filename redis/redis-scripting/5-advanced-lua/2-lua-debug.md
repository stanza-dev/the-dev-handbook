---
source_course: "redis-scripting"
source_lesson: "redis-scripting-lua-debug"
---

# Script Flags and Debugging

Optimize and debug your Lua scripts for production use.

## Script Flags

Redis 7.0+ supports script flags for better control:

```lua
#!lua flags=no-writes
-- This script is read-only and can run on replicas

return redis.call('GET', KEYS[1])
```

### Available Flags

| Flag | Description |
|------|-------------|
| `no-writes` | Script doesn't write data |
| `allow-oom` | Run even when memory limit reached |
| `allow-stale` | Run on stale replica |
| `no-cluster` | Disable in cluster mode |
| `allow-cross-slot-keys` | Allow keys from different slots |

```lua
#!lua flags=no-writes,allow-stale
-- Read-only script that accepts stale data

local value = redis.call('GET', KEYS[1])
return value or 'default'
```

## Error Handling

```lua
-- Good error handling pattern
local function safe_get_number(key)
    local value = redis.call('GET', key)
    if value == false then
        return nil, 'KEY_NOT_FOUND'
    end
    local num = tonumber(value)
    if num == nil then
        return nil, 'NOT_A_NUMBER'
    end
    return num, nil
end

local balance, err = safe_get_number(KEYS[1])
if err then
    return redis.error_reply(err)
end

-- Continue with balance...
```

## redis.pcall for Error Recovery

```lua
-- Try operation, handle failure gracefully
local result = redis.pcall('INCR', KEYS[1])

if type(result) == 'table' and result.err then
    -- INCR failed (probably not a number)
    redis.call('SET', KEYS[1], 1)
    return 1
end

return result
```

## Debugging Techniques

### Using redis.log

```lua
-- Log at different levels
redis.log(redis.LOG_DEBUG, 'Debug message')
redis.log(redis.LOG_VERBOSE, 'Verbose message')
redis.log(redis.LOG_NOTICE, 'Notice message')
redis.log(redis.LOG_WARNING, 'Warning message')

-- Example: Debug complex logic
local balance = redis.call('GET', KEYS[1])
redis.log(redis.LOG_DEBUG, 'Balance for ' .. KEYS[1] .. ': ' .. tostring(balance))
```

### Script Debugging Mode

```bash
# Start Redis CLI in debug mode
redis-cli --ldb --eval myscript.lua key1 key2 , arg1 arg2

# Debug commands:
# s - step into
# n - step over
# c - continue
# p <var> - print variable
# b <line> - set breakpoint
# w - where (show stack)
# e <code> - evaluate Lua code
```

## Performance Best Practices

```lua
-- BAD: Creating tables in hot path
for i = 1, 1000 do
    local result = {}  -- Allocates memory each iteration
    table.insert(result, i)
end

-- GOOD: Reuse tables
local result = {}
for i = 1, 1000 do
    result[i] = i
end

-- BAD: String concatenation in loops
local str = ''
for i = 1, 100 do
    str = str .. redis.call('GET', 'key:' .. i)  -- Creates new string each time
end

-- GOOD: Use table.concat
local parts = {}
for i = 1, 100 do
    parts[i] = redis.call('GET', 'key:' .. i)
end
local str = table.concat(parts, '')
```

## Script Management

```redis
# Check if script exists
SCRIPT EXISTS <sha1> [sha1 ...]

# Kill running script
SCRIPT KILL

# Clear script cache
SCRIPT FLUSH

# Debug mode
SCRIPT DEBUG YES|SYNC|NO
```

📖 [Script Commands](https://redis.io/docs/latest/commands/?group=scripting)

## Resources

- [Scripting Commands](https://redis.io/docs/latest/commands/?group=scripting) — Redis scripting command reference

---

> 📘 *This lesson is part of the [Scripting, Transactions, and Programmability](https://stanza.dev/courses/redis-scripting) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*