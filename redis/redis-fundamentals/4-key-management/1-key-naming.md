---
source_course: "redis-fundamentals"
source_lesson: "redis-fundamentals-key-naming"
---

# Key Naming Conventions

Good key naming is crucial for maintaining organized, scalable Redis databases. Poor naming leads to confusion and makes it hard to manage data.

## Best Practices

### Use Colons as Separators

The Redis community convention uses colons to create namespaces:

```redis
# Good: Clear hierarchical structure
user:1001:profile
user:1001:sessions
user:1001:preferences
order:5001:items
order:5001:status

# Bad: Inconsistent, hard to parse
user_1001_profile
user.1001.profile
User1001Profile
```

### Pattern: object-type:id:field

```redis
# Users
user:1001              # User hash
user:1001:sessions     # User's active sessions
user:1001:followers    # User's followers set

# Products
product:SKU123         # Product hash
product:SKU123:reviews # Product reviews

# Sessions
session:abc123xyz      # Session data
```

### Use Meaningful Prefixes

```redis
# Different data types, clear purpose
cache:api:users:list      # Cached API response
queue:emails:pending      # Email queue
lock:order:5001           # Distributed lock
rate:api:user:1001        # Rate limit counter
temp:import:batch:42      # Temporary processing data
```

## Environment Separation

```redis
# Option 1: Prefix with environment
dev:user:1001
staging:user:1001
prod:user:1001

# Option 2: Use different databases (0-15)
SELECT 0    # Production
SELECT 1    # Staging
SELECT 2    # Development
```

## Key Length Considerations

- Keys are stored in memory, so shorter is better
- But readability matters more than a few bytes
- Avoid extremely long keys (>1KB)

```redis
# Good balance
user:1001:cart

# Too verbose
application:ecommerce:customer:user_id:1001:shopping_cart:items

# Too cryptic
u:1001:c
```

## Scanning for Keys

### KEYS Command (Use with Caution!)

```redis
# Find all user keys
KEYS user:*

# Find all sessions
KEYS session:*

# ⚠️ WARNING: KEYS blocks Redis!
# Never use in production on large databases
```

### SCAN Command (Production Safe)

```redis
# Iterate through keys safely
SCAN 0 MATCH user:* COUNT 100
# Returns: cursor and matching keys

# Continue scanning
SCAN <cursor> MATCH user:* COUNT 100
# Repeat until cursor is 0
```

**SCAN benefits:**
- Non-blocking (iterative)
- Returns results incrementally
- Safe for production

## Checking Key Information

```redis
# Get type of key
TYPE user:1001
# Returns: hash, string, list, set, zset, etc.

# Check if key exists
EXISTS user:1001
# Returns: 1 or 0

# Check multiple keys
EXISTS user:1001 user:1002 user:1003
# Returns: count of existing keys

# Get memory usage of key
MEMORY USAGE user:1001
# Returns: bytes used
```

📖 [Redis Keys Documentation](https://redis.io/docs/latest/commands/?group=generic)

---

> 📘 *This lesson is part of the [In-Memory Data Store Fundamentals](https://stanza.dev/courses/redis-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*