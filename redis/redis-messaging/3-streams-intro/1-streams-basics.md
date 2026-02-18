---
source_course: "redis-messaging"
source_lesson: "redis-messaging-streams-basics"
---

# Redis Streams Overview

Redis Streams is an append-only log data structure that addresses the limitations of Pub/Sub. Think of it as a persistent message queue with powerful consumer features.

## Streams vs Pub/Sub

| Feature | Pub/Sub | Streams |
|---------|---------|--------|
| Persistence | No | Yes |
| Message history | No | Yes |
| Acknowledgment | No | Yes |
| Consumer groups | No | Yes |
| Replay messages | No | Yes |
| Blocking reads | Yes | Yes |

## Stream Structure

```
┌─────────────────────────────────────────────────────────────┐
│  Stream: orders                                             │
│                                                             │
│  ID                  │  Fields                              │
│  ────────────────────┼─────────────────────────────────────│
│  1704067200000-0     │  customer: alice, total: 99.99     │
│  1704067200001-0     │  customer: bob, total: 149.50      │
│  1704067200002-0     │  customer: charlie, total: 50.00   │
│                      │                                     │
│  ← Append only (oldest)                    (newest) →      │
└─────────────────────────────────────────────────────────────┘
```

## Entry IDs

Each stream entry has a unique ID in format: `<millisecondsTime>-<sequenceNumber>`

```
1704067200000-0  →  First entry at timestamp 1704067200000
1704067200000-1  →  Second entry at same millisecond
1704067200001-0  →  First entry at next millisecond
```

## XADD: Adding Entries

```redis
# Auto-generate ID with *
XADD orders * customer alice total 99.99 status pending
# Returns: "1704067200000-0"

# Add multiple entries
XADD orders * customer bob total 149.50 status pending
XADD orders * customer charlie total 50.00 status pending

# Specify custom ID (must be greater than last)
XADD orders 1704067300000-0 customer dave total 200.00
```

## XLEN: Stream Length

```redis
XLEN orders
# Returns: (integer) 4
```

## XRANGE: Reading by ID Range

```redis
# Get all entries
XRANGE orders - +
# - means minimum ID, + means maximum ID

# Get specific range
XRANGE orders 1704067200000-0 1704067200002-0

# Get first 2 entries
XRANGE orders - + COUNT 2

# Get entries after a specific ID
XRANGE orders 1704067200001-0 + COUNT 10
```

## XREVRANGE: Reverse Order

```redis
# Get latest entries first
XREVRANGE orders + - COUNT 5
# Returns 5 most recent entries
```

## XREAD: Blocking Reads

```redis
# Read new entries (non-blocking)
XREAD STREAMS orders 0
# 0 = from beginning

# Read entries after specific ID
XREAD STREAMS orders 1704067200001-0

# Block until new data arrives
XREAD BLOCK 5000 STREAMS orders $
# $ = only new entries
# BLOCK 5000 = wait up to 5000ms

# Read from multiple streams
XREAD BLOCK 0 STREAMS orders notifications $ $
```

## Stream Entry Structure

Entries are field-value pairs (like a hash):

```redis
XADD events * \
    type "user_signup" \
    user_id "1001" \
    email "alice@example.com" \
    timestamp "2024-01-01T10:00:00Z"
```

## Practical Example: Event Log

```redis
# Log application events
XADD app:events * \
    level "INFO" \
    service "auth" \
    message "User login successful" \
    user_id "1001"

XADD app:events * \
    level "ERROR" \
    service "payment" \
    message "Payment failed" \
    order_id "5001" \
    error "Card declined"

# Query recent errors
XREVRANGE app:events + - COUNT 100
```

📖 [Redis Streams](https://redis.io/docs/latest/develop/data-types/streams/)

## Resources

- [Redis Streams](https://redis.io/docs/latest/develop/data-types/streams/) — Complete guide to Redis Streams

---

> 📘 *This lesson is part of the [Real-Time Messaging and Event Streaming](https://stanza.dev/courses/redis-messaging) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*