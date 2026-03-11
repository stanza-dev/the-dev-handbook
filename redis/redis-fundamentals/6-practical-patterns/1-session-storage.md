---
source_course: "redis-fundamentals"
source_lesson: "redis-fundamentals-session-storage"
---

# Session Storage Pattern

## Introduction

Redis is the industry standard for storing web application sessions. Its sub-millisecond access, built-in expiration, and ability to be shared across multiple app servers make it far superior to file-based or in-memory session stores.

## Key Concepts

- **Session as hash**: Store session fields (user_id, role, created_at) as a Redis hash with HSET.
- **Sliding expiration**: Call EXPIRE on every request to reset the TTL, keeping active sessions alive.
- **Multi-session tracking**: Use a Set per user to track all their active sessions for "logout everywhere" functionality.
- **JSON sessions**: Redis JSON allows storing complex nested session data with atomic sub-field updates.

## Real World Context

Every multi-server web application needs shared sessions. Without Redis, sticky sessions or database-backed sessions create scaling bottlenecks. Redis sessions enable horizontal scaling — add more app servers without session affinity.

## Deep Dive

Redis is perfect for storing user sessions. It's fast, supports automatic expiration, and can be shared across multiple application servers.

## Why Redis for Sessions?

```
┌─────────────────────────────────────────────────────────┐
│  Traditional File Sessions:                              │
│  - Tied to one server                                   │
│  - Slow disk I/O                                        │
│  - Hard to scale                                        │
├─────────────────────────────────────────────────────────┤
│  Redis Sessions:                                         │
│  - Shared across all servers                            │
│  - Sub-millisecond access                               │
│  - Built-in expiration                                  │
│  - Easy horizontal scaling                              │
└─────────────────────────────────────────────────────────┘
```

## Simple Session Implementation

### Creating a Session

```redis
# Generate session ID in your app (UUID recommended)
# session_id = "sess:a1b2c3d4-e5f6-7890-abcd-ef1234567890"

# Store session data as hash
HSET sess:a1b2c3d4 \
    user_id 1001 \
    email "alice@example.com" \
    role "admin" \
    created_at 1704067200 \
    last_activity 1704067200

# Set expiration (30 minutes)
EXPIRE sess:a1b2c3d4 1800
```

### Reading Session

```redis
# Get all session data
HGETALL sess:a1b2c3d4

# Get specific fields
HMGET sess:a1b2c3d4 user_id role

# Check if session exists
EXISTS sess:a1b2c3d4
```

### Refreshing Session (Sliding Expiration)

```redis
# On each request, refresh expiration
EXPIRE sess:a1b2c3d4 1800

# Update last activity
HSET sess:a1b2c3d4 last_activity 1704070800
```

### Destroying Session

```redis
# Logout - delete session
DEL sess:a1b2c3d4
```

## JSON Session Alternative

```redis
# Store session as JSON
JSON.SET sess:a1b2c3d4 $ '{
    "user_id": 1001,
    "email": "alice@example.com",
    "role": "admin",
    "permissions": ["read", "write", "admin"],
    "preferences": {
        "theme": "dark",
        "language": "en"
    }
}'
EXPIRE sess:a1b2c3d4 1800

# Update nested field
JSON.SET sess:a1b2c3d4 $.preferences.theme '"light"'

# Read specific path
JSON.GET sess:a1b2c3d4 $.user_id $.role
```

## Managing Multiple Sessions

```redis
# Track all sessions for a user (for "logout everywhere")
SADD user:1001:sessions "sess:a1b2c3d4" "sess:e5f6g7h8"

# Logout everywhere
SMEMBERS user:1001:sessions
# For each session_id in result:
DEL sess:a1b2c3d4
DEL sess:e5f6g7h8
DEL user:1001:sessions
```

## Session Security Tips

1. **Use secure random session IDs** (UUIDs or crypto-random strings)
2. **Set appropriate TTLs** (15-60 minutes for sensitive apps)
3. **Bind sessions to IP/User-Agent** for extra security
4. **Invalidate on password change** using the multi-session pattern
5. **Use HTTPS** to protect session cookies in transit

📖 [Redis as Session Store](https://redis.io/docs/latest/develop/use/patterns/)

## Common Pitfalls

1. **Not setting session expiration** — Sessions without TTL accumulate forever, wasting memory. Always set EXPIRE.
2. **Using predictable session IDs** — Never use sequential or guessable IDs. Generate cryptographically random UUIDs or tokens.

## Best Practices

1. **Use sliding expiration** — Reset TTL on each request so active users stay logged in while idle sessions expire naturally.
2. **Track sessions per user** — Maintain a Set of active session IDs per user to enable "logout all devices" and detect session theft.

## Summary

- Store sessions as Redis hashes with HSET for field-level access.
- Use EXPIRE with sliding window to auto-expire inactive sessions.
- Track all user sessions in a Set for "logout everywhere" support.
- Redis JSON is an alternative for complex nested session data.
- Always use cryptographically random session IDs.

## Code Examples

**Basic Redis session lifecycle — create with HSET, set TTL with EXPIRE, refresh on activity, and delete on logout**

```bash
# Create session with hash + expiration
HSET sess:a1b2c3d4 user_id 1001 email "alice@example.com" role "admin"
EXPIRE sess:a1b2c3d4 1800

# Sliding expiration: refresh on each request
EXPIRE sess:a1b2c3d4 1800

# Logout: destroy session
DEL sess:a1b2c3d4
```


## Resources

- [Redis Patterns](https://redis.io/docs/latest/develop/use/patterns/) — Common Redis usage patterns including session storage

---

> 📘 *This lesson is part of the [In-Memory Data Store Fundamentals](https://stanza.dev/courses/redis-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*