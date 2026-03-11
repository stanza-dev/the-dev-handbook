---
source_course: "redis-fundamentals"
source_lesson: "redis-fundamentals-bitmaps"
---

# Bitmaps: Efficient Binary Data

## Introduction

Bitmaps are Redis strings treated as arrays of bits, enabling extremely memory-efficient tracking of binary states across millions of items. Where a Set of user IDs might use megabytes, a bitmap achieves the same tracking in kilobytes.

## Key Concepts

- **SETBIT/GETBIT**: Set or read a single bit at a given offset (position).
- **BITCOUNT**: Count the number of bits set to 1 — perfect for counting active users.
- **BITOP**: Perform AND, OR, XOR, NOT across multiple bitmaps to combine datasets.
- **Memory efficiency**: 1 million users tracked in a bitmap uses only 125 KB vs 8 MB in a Set.

## Real World Context

Bitmaps power daily/weekly/monthly active user (DAU/WAU/MAU) analytics, feature flag systems, user retention analysis (BITOP AND across days), and bloom-filter-like duplicate detection. Any binary per-user state is a bitmap candidate.

## Deep Dive

Bitmaps are not a separate data type - they're strings treated as arrays of bits. They're incredibly memory-efficient for tracking binary states across millions of items.

## Why Bitmaps?

```
┌─────────────────────────────────────────────────────────────┐
│  Problem: Track which users logged in today                 │
│                                                             │
│  Set approach: Store user IDs                              │
│  - 1 million users × 8 bytes = 8 MB                        │
│                                                             │
│  Bitmap approach: 1 bit per user                           │
│  - 1 million users × 1 bit = 125 KB                        │
│  - 64x more efficient!                                      │
└─────────────────────────────────────────────────────────────┘
```

## Basic Commands

### SETBIT and GETBIT

```redis
# Set bit at offset (0-indexed)
SETBIT logins:2024-01-15 1001 1   # User 1001 logged in
SETBIT logins:2024-01-15 1002 1   # User 1002 logged in
SETBIT logins:2024-01-15 1003 1   # User 1003 logged in

# Check if user logged in
GETBIT logins:2024-01-15 1001     # Returns: 1 (logged in)
GETBIT logins:2024-01-15 9999     # Returns: 0 (didn't log in)
```

### BITCOUNT: Count Set Bits

```redis
# Count how many users logged in
BITCOUNT logins:2024-01-15
# Returns: 3

# Count bits in byte range
BITCOUNT mykey 0 0    # First byte only
BITCOUNT mykey 0 -1   # All bytes
```

### BITPOS: Find First Bit

```redis
# Find first user who logged in
BITPOS logins:2024-01-15 1    # Returns: 1001 (first set bit)

# Find first user who didn't log in (after offset)
BITPOS logins:2024-01-15 0 0  # Returns: 0 (first unset bit)
```

## Bitwise Operations

Perform operations across multiple bitmaps:

```redis
# Set up test data
SETBIT logins:day1 1 1
SETBIT logins:day1 2 1
SETBIT logins:day1 3 1

SETBIT logins:day2 2 1
SETBIT logins:day2 3 1
SETBIT logins:day2 4 1

# Users who logged in BOTH days (AND)
BITOP AND logins:both_days logins:day1 logins:day2
BITCOUNT logins:both_days    # Returns: 2 (users 2 and 3)

# Users who logged in EITHER day (OR)
BITOP OR logins:either_day logins:day1 logins:day2
BITCOUNT logins:either_day   # Returns: 4 (users 1,2,3,4)

# Users who logged in day1 but NOT day2 (XOR then AND)
BITOP XOR logins:xor logins:day1 logins:day2

# Invert bitmap (NOT)
BITOP NOT logins:not_day1 logins:day1
```

## Practical Examples

### Daily Active Users (DAU)

```python
import redis
from datetime import datetime, timedelta

r = redis.Redis()

def mark_user_active(user_id):
    today = datetime.now().strftime('%Y-%m-%d')
    r.setbit(f'active:{today}', user_id, 1)

def get_dau(date):
    return r.bitcount(f'active:{date}')

def get_wau():
    """Weekly Active Users - users active in last 7 days"""
    keys = []
    for i in range(7):
        date = (datetime.now() - timedelta(days=i)).strftime('%Y-%m-%d')
        keys.append(f'active:{date}')
    
    # OR all days together
    r.bitop('OR', 'active:week', *keys)
    return r.bitcount('active:week')

def get_retention(date1, date2):
    """Users active on both dates"""
    r.bitop('AND', 'retention:temp', f'active:{date1}', f'active:{date2}')
    return r.bitcount('retention:temp')
```

### Feature Flags per User

```redis
# Each bit represents a feature
# Bit 0: Dark mode
# Bit 1: Beta features
# Bit 2: Premium

# Enable dark mode for user 1001
SETBIT features:user:1001 0 1

# Enable premium for user 1001
SETBIT features:user:1001 2 1

# Check if user has premium
GETBIT features:user:1001 2    # Returns: 1
```

### Bloom Filter Alternative

For simple presence checking:

```redis
# Track seen emails (using hash of email as offset)
# Note: This can have collisions, similar to Bloom filter
SETBIT seen_emails [hash_of_email % 10000000] 1
```

## Memory Efficiency

```
Users    Set (IDs)    Bitmap      Savings
───────────────────────────────────────────
1K       8 KB         128 bytes   98%
100K     800 KB       12.5 KB     98%
1M       8 MB         125 KB      98%
10M      80 MB        1.25 MB     98%
```

## Limitations

- Maximum offset: 2^32 - 1 (about 4 billion)
- Sparse bitmaps waste memory (user ID 1 and 1 billion)
- Use Roaring Bitmaps for sparse data

📖 [Bitmaps Documentation](https://redis.io/docs/latest/develop/data-types/bitmaps/)

## Common Pitfalls

1. **Sparse bitmaps waste memory** — If your user IDs are 1 and 1,000,000,000, the bitmap allocates space for all bits in between. Use Sets or Roaring Bitmaps for sparse data.
2. **Confusing byte range with bit range in BITCOUNT** — BITCOUNT's optional range parameters operate on bytes, not bits. BITCOUNT key 0 0 counts bits in the first byte only.

## Best Practices

1. **Use sequential IDs as offsets** — Bitmaps are most efficient when IDs are dense and sequential. Map user IDs to compact offsets if needed.
2. **Combine daily bitmaps with BITOP** — OR for "active any day this week" (WAU), AND for "active every day this week" (retention).

## Summary

- Bitmaps are strings treated as bit arrays — 64x more memory efficient than Sets for binary tracking.
- SETBIT marks a user; BITCOUNT counts how many are marked.
- BITOP AND/OR/XOR/NOT combines multiple bitmaps for retention and engagement analysis.
- Maximum offset is 2^32-1 (about 4 billion).

## Code Examples

**Bitmaps for analytics — SETBIT marks users, BITCOUNT counts them, BITOP AND finds users active on multiple days**

```bash
# Track daily active users (1 bit per user)
SETBIT active:2024-01-15 1001 1
SETBIT active:2024-01-15 1002 1
BITCOUNT active:2024-01-15
# (integer) 2

# Users active on BOTH days (AND)
BITOP AND active:both active:day1 active:day2
BITCOUNT active:both
```


## Resources

- [Redis Bitmaps](https://redis.io/docs/latest/develop/data-types/bitmaps/) — Guide to Redis Bitmaps

---

> 📘 *This lesson is part of the [In-Memory Data Store Fundamentals](https://stanza.dev/courses/redis-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*