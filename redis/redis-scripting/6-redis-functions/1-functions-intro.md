---
source_course: "redis-scripting"
source_lesson: "redis-scripting-functions-intro"
---

# Redis Functions Overview

Redis Functions (introduced in Redis 7.0) provide a modern, persistent alternative to EVAL scripts. Functions are stored in the database and persist across restarts.

## Functions vs EVAL Scripts

| Feature | EVAL Scripts | Functions |
|---------|-------------|------------|
| Persistence | No (client must reload) | Yes (stored in Redis) |
| Loading | Every execution or SCRIPT LOAD | Once per deployment |
| Organization | Individual scripts | Libraries of functions |
| Cluster | Must use KEYS[] | Same, but clearer API |
| Replication | Not replicated | Replicated to replicas |

## Creating a Function Library

```lua
#!lua name=mylib

-- Helper function (not registered)
local function validate_balance(balance)
    return balance and tonumber(balance) >= 0
end

-- Registered function
redis.register_function('get_balance', function(keys, args)
    local balance = redis.call('GET', keys[1])
    if validate_balance(balance) then
        return balance
    else
        return '0'
    end
end)

-- Another function in same library
redis.register_function('transfer', function(keys, args)
    local from_key = keys[1]
    local to_key = keys[2]
    local amount = tonumber(args[1])
    
    local from_balance = tonumber(redis.call('GET', from_key)) or 0
    
    if from_balance < amount then
        return redis.error_reply('Insufficient funds')
    end
    
    redis.call('DECRBY', from_key, amount)
    redis.call('INCRBY', to_key, amount)
    
    return 'OK'
end)
```

## Loading Functions

```redis
# Load the library
FUNCTION LOAD "#!lua name=mylib\n\nredis.register_function('hello', function() return 'Hello!' end)"

# Load from file (using redis-cli)
cat mylib.lua | redis-cli -x FUNCTION LOAD

# Replace existing library
FUNCTION LOAD REPLACE "#!lua name=mylib ..."
```

## Calling Functions

```redis
# Basic call
FCALL get_balance 1 user:1001:balance

# With arguments
FCALL transfer 2 user:1001:balance user:1002:balance 50

# Read-only variant (can run on replicas)
FCALL_RO get_balance 1 user:1001:balance
```

## Managing Functions

```redis
# List all functions
FUNCTION LIST

# List with library code
FUNCTION LIST WITHCODE

# Delete a library
FUNCTION DELETE mylib

# Get statistics
FUNCTION STATS

# Delete all functions
FUNCTION FLUSH
```

## Function Flags

```lua
#!lua name=mylib

-- Read-only function (can run on replicas, during loading)
redis.register_function{
    function_name = 'get_user',
    callback = function(keys, args)
        return redis.call('HGETALL', keys[1])
    end,
    flags = {'no-writes'}
}

-- Allow stale reads
redis.register_function{
    function_name = 'cached_get',
    callback = function(keys, args)
        return redis.call('GET', keys[1])
    end,
    flags = {'no-writes', 'allow-stale'}
}
```

## Available Flags

- `no-writes`: Function doesn't write data
- `allow-oom`: Allow execution even when OOM
- `allow-stale`: Allow on stale replica
- `no-cluster`: Disallow in cluster mode

## Practical Example: User Service

```lua
#!lua name=userservice

redis.register_function{
    function_name = 'create_user',
    callback = function(keys, args)
        local user_id = args[1]
        local email = args[2]
        local name = args[3]
        
        local user_key = 'user:' .. user_id
        local email_key = 'email:' .. email
        
        -- Check if email exists
        if redis.call('EXISTS', email_key) == 1 then
            return redis.error_reply('Email already registered')
        end
        
        -- Create user
        redis.call('HSET', user_key,
            'id', user_id,
            'email', email,
            'name', name,
            'created_at', redis.call('TIME')[1]
        )
        
        -- Index email
        redis.call('SET', email_key, user_id)
        
        return 'OK'
    end
}

redis.register_function{
    function_name = 'get_user_by_email',
    callback = function(keys, args)
        local email = args[1]
        local user_id = redis.call('GET', 'email:' .. email)
        
        if not user_id then
            return nil
        end
        
        return redis.call('HGETALL', 'user:' .. user_id)
    end,
    flags = {'no-writes'}
}
```

```redis
# Use the functions
FCALL create_user 0 1001 alice@example.com Alice
FCALL_RO get_user_by_email 0 alice@example.com
```

📖 [Redis Functions](https://redis.io/docs/latest/develop/interact/programmability/functions-intro/)

## Resources

- [Redis Functions](https://redis.io/docs/latest/develop/interact/programmability/functions-intro/) — Guide to Redis Functions

---

> 📘 *This lesson is part of the [Scripting, Transactions, and Programmability](https://stanza.dev/courses/redis-scripting) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*