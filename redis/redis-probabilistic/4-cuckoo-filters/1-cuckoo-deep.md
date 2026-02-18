---
source_course: "redis-probabilistic"
source_lesson: "redis-probabilistic-cuckoo-deep"
---

# Cuckoo Filters: Bloom with Deletion

Cuckoo filters are similar to Bloom filters but support item deletion. They're slightly larger but more flexible.

## Bloom vs Cuckoo

| Feature | Bloom Filter | Cuckoo Filter |
|---------|-------------|---------------|
| Add items | Yes | Yes |
| Check membership | Yes | Yes |
| Delete items | No | Yes |
| Memory | Smaller | Slightly larger |
| False positives | Yes | Yes |
| Count occurrences | No | Limited |

## When to Use Cuckoo

- Need to remove items from the filter
- Tracking temporary membership (sessions, reservations)
- Building expiring caches with membership check

## Basic Commands

### CF.RESERVE: Create Filter

```redis
# Create Cuckoo filter
CF.RESERVE sessions 1000000
# Capacity of 1 million items

# With bucket size (affects accuracy)
CF.RESERVE sessions 1000000 BUCKETSIZE 4
```

### CF.ADD: Add Items

```redis
# Add item
CF.ADD sessions "session:abc123"
# Returns: 1 (success) or error if filter is full

# Add if not exists
CF.ADDNX sessions "session:abc123"
# Returns: 0 (already might exist), 1 (added)
```

### CF.EXISTS: Check Membership

```redis
# Check single item
CF.EXISTS sessions "session:abc123"
# Returns: 1 (probably exists) or 0 (definitely not)

# Check multiple
CF.MEXISTS sessions "session:abc123" "session:xyz789"
```

### CF.DEL: Delete Items

```redis
# Delete item
CF.DEL sessions "session:abc123"
# Returns: 1 (deleted) or 0 (didn't exist)
```

## How Cuckoo Filters Work

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
└─────────────────────────────────────────────────────────────┘
```

📖 [Cuckoo Filter Commands](https://redis.io/docs/latest/develop/data-types/probabilistic/cuckoo-filter/)

## Resources

- [Cuckoo Filters](https://redis.io/docs/latest/develop/data-types/probabilistic/cuckoo-filter/) — Redis Cuckoo Filter documentation

---

> 📘 *This lesson is part of the [Probabilistic Data Structures and Analytics](https://stanza.dev/courses/redis-probabilistic) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*