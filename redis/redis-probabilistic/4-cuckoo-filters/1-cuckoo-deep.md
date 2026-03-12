---
source_course: "redis-probabilistic"
source_lesson: "redis-probabilistic-cuckoo-deep"
---

# Cuckoo Filters: Bloom with Deletion

## Introduction

Cuckoo filters solve the biggest limitation of Bloom filters: the inability to delete items. If your application tracks temporary state — sessions, reservations, feature flags — and needs to remove entries without rebuilding the entire filter, Cuckoo filters are your tool.

## Key Concepts

- **Fingerprint**: A compact hash of the original item, stored in a bucket (typically 1-4 bytes)
- **Bucket**: A fixed-size slot array where fingerprints are stored; each bucket holds multiple fingerprints
- **Cuckoo Hashing**: When a bucket is full, an existing fingerprint is "kicked" to its alternate bucket (like a cuckoo bird evicting eggs from a nest)
- **Alternate Location**: Each fingerprint has exactly two possible bucket locations, computed via `h1 XOR hash(fingerprint)`
- **Deletion**: Because fingerprints can be located and removed, Cuckoo filters support CF.DEL — something Bloom filters cannot offer

## Real World Context

Imagine a session management system for a live-streaming platform. Users log in and out throughout the day. With a Bloom filter, you can add sessions but never remove them — the filter fills up with stale sessions and the false positive rate climbs. A Cuckoo filter lets you CF.DEL a session on logout, keeping the filter accurate without periodic rebuilds.

## Deep Dive

### Bloom vs Cuckoo Comparison

| Feature | Bloom Filter | Cuckoo Filter |
|---------|-------------|---------------|
| Add items | Yes | Yes |
| Check membership | Yes | Yes |
| Delete items | No | Yes |
| Memory | Smaller | Slightly larger |
| False positives | Yes | Yes |
| Count occurrences | No | Approximate |

### How Cuckoo Filters Work

```
┌─────────────────────────────────────────────────────────────┐
│  Cuckoo Filter uses 2 hash functions and "nests" (buckets) │
│                                                             │
│  Add "apple":                                              │
│  - fingerprint = hash("apple") = 0xAB                     │
│  - h1 = hash1("apple") = 3                                │
│  - h2 = h1 XOR hash(fingerprint) = 7                      │
│                                                             │
│  Store fingerprint 0xAB in bucket 3 or 7                   │
│                                                             │
│  If both full: "kick" existing item to its alternate bucket│
│  (Like a cuckoo bird kicking eggs from nests)              │
│                                                             │
│  Delete "apple":                                           │
│  - Compute fingerprint and both bucket locations           │
│  - Find 0xAB in bucket 3 or 7 and remove it               │
└─────────────────────────────────────────────────────────────┘
```

### Core Commands

#### CF.RESERVE: Create Filter

```redis
# Create Cuckoo filter with capacity
CF.RESERVE sessions 1000000

# With bucket size (affects accuracy and memory)
CF.RESERVE sessions 1000000 BUCKETSIZE 4
```

#### CF.ADD / CF.ADDNX: Add Items

```redis
# Add item (allows duplicates)
CF.ADD sessions "session:abc123"
# Returns: 1 (success)

# Add only if not already present
CF.ADDNX sessions "session:abc123"
# Returns: 0 (already might exist), 1 (added)
```

#### CF.EXISTS / CF.MEXISTS: Check Membership

```redis
# Check single item
CF.EXISTS sessions "session:abc123"
# Returns: 1 (probably exists) or 0 (definitely not)

# Check multiple
CF.MEXISTS sessions "session:abc123" "session:xyz789"
```

#### CF.DEL: Delete Items

```redis
# Delete item
CF.DEL sessions "session:abc123"
# Returns: 1 (deleted) or 0 (didn't exist)
```

#### CF.COUNT: Approximate Count

```redis
CF.ADD items "apple"
CF.ADD items "apple"
CF.ADD items "apple"

CF.COUNT items "apple"
# Returns: 3 (approximate)
```

### When to Choose Cuckoo Over Bloom

| Scenario | Recommendation |
|----------|----------------|
| Write-once data (URLs, IDs) | Bloom Filter |
| Temporary data (sessions) | Cuckoo Filter |
| Need deletion | Cuckoo Filter |
| Minimum memory | Bloom Filter |
| Need approximate counting | Cuckoo Filter |

## Common Pitfalls

1. **Deleting items that were never added** — CF.DEL can remove a fingerprint that belongs to a different item with the same fingerprint, causing future false negatives. Always check CF.EXISTS before calling CF.DEL.
2. **Assuming unlimited capacity** — Unlike Bloom filters that degrade gracefully, a Cuckoo filter can fail (return an error) when it runs out of space after too many kicks. Monitor with CF.INFO and provision adequate capacity.

## Best Practices

1. **Use CF.ADDNX for idempotent inserts** — If your application might add the same item multiple times, CF.ADDNX prevents duplicate fingerprints from accumulating and inflating CF.COUNT results.
2. **Pair CF.DEL with authoritative state** — Always have a source of truth (database, hash) alongside the Cuckoo filter. Use the filter for fast checks but verify against the source of truth for critical operations.

## Summary

- Cuckoo filters store fingerprints in buckets with two possible locations per item
- They support deletion via CF.DEL, which Bloom filters cannot do
- CF.ADD adds items, CF.EXISTS checks membership, CF.COUNT gives approximate occurrence counts
- Use Cuckoo filters when items are temporary or need removal; use Bloom filters for append-only data with minimal memory

## Code Examples

**Session management with Cuckoo filter for fast validity checks and clean deletion on logout**

```python
import redis
import uuid
from datetime import datetime

r = redis.Redis()

class SessionManager:
    def __init__(self, capacity=1000000):
        self.filter_key = 'active:sessions'
        try:
            r.execute_command('CF.RESERVE', self.filter_key, capacity)
        except redis.ResponseError:
            pass

    def create_session(self, user_id):
        """Create a new session."""
        session_id = str(uuid.uuid4())
        r.hset(f'session:{session_id}', mapping={
            'user_id': user_id,
            'created': str(datetime.now())
        })
        r.expire(f'session:{session_id}', 3600)
        r.execute_command('CF.ADD', self.filter_key, session_id)
        return session_id

    def is_valid_session(self, session_id):
        """Quick check if session might be valid."""
        if not r.execute_command('CF.EXISTS', self.filter_key, session_id):
            return False  # Definitely invalid
        return r.exists(f'session:{session_id}')  # Confirm with source of truth

    def destroy_session(self, session_id):
        """Remove session from both storage and filter."""
        r.delete(f'session:{session_id}')
        r.execute_command('CF.DEL', self.filter_key, session_id)
```


## Resources

- [Cuckoo Filters](https://redis.io/docs/latest/develop/data-types/probabilistic/cuckoo-filter/) — Redis Cuckoo Filter documentation
- [CF.RESERVE Command](https://redis.io/docs/latest/commands/cf.reserve/) — CF.RESERVE command reference

---

> 📘 *This lesson is part of the [Probabilistic Data Structures and Analytics](https://stanza.dev/courses/redis-probabilistic) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*