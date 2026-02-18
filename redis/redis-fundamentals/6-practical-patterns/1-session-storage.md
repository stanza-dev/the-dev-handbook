---
source_course: "redis-fundamentals"
source_lesson: "redis-fundamentals-session-storage"
---

# Session Storage Pattern

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

---

> 📘 *This lesson is part of the [In-Memory Data Store Fundamentals](https://stanza.dev/courses/redis-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*