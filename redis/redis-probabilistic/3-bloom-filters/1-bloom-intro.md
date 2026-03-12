---
source_course: "redis-probabilistic"
source_lesson: "redis-probabilistic-bloom-intro"
---

# Understanding Bloom Filters

## Introduction

A Bloom filter is a space-efficient probabilistic data structure that answers one question: "Is this item in the set?" It can definitively tell you an item is NOT present, but can only say an item is PROBABLY present. This trade-off buys you dramatic memory savings compared to storing every item.

## Key Concepts

- **Bit Array**: The underlying storage — a fixed-size array of bits initialized to 0
- **Hash Functions**: Multiple independent hash functions that map items to bit positions
- **False Positive**: When the filter says "yes" but the item was never added (due to hash collisions)
- **False Negative**: When the filter says "no" but the item was added — this NEVER happens with Bloom filters
- **Error Rate**: The probability of a false positive, configurable at creation time
- **Capacity**: The expected number of items the filter will hold

## Real World Context

Every time a user hits your API, you might need to check "has this request been seen before?" or "is this username taken?" With millions of items, storing them all in a Set costs gigabytes. A Bloom filter answers the same question in megabytes. Google Chrome used Bloom filters to check URLs against a malware list — testing millions of URLs locally without downloading the entire database.

## Deep Dive

### How It Works

```
┌─────────────────────────────────────────────────────────────┐
│  Bit Array (initialized to 0):                              │
│  [0][0][0][0][0][0][0][0][0][0]                            │
│                                                             │
│  Add "apple":                                              │
│  - hash1("apple") = 2                                      │
│  - hash2("apple") = 5                                      │
│  - hash3("apple") = 7                                      │
│  [0][0][1][0][0][1][0][1][0][0]                            │
│                                                             │
│  Check "apple": Bits 2,5,7 all set? Yes → Probably exists │
│  Check "banana": Bits 1,3,8 set? No → Definitely not exists│
└─────────────────────────────────────────────────────────────┘
```

### Core Commands

#### BF.RESERVE: Create a Filter

```redis
# Create a filter with error rate and capacity
BF.RESERVE users 0.01 1000000
# 0.01 = 1% false positive rate
# 1000000 = expected number of items

# With expansion (auto-grow when capacity exceeded)
BF.RESERVE users 0.01 1000000 EXPANSION 2
```

#### BF.ADD / BF.MADD: Add Items

```redis
# Add single item
BF.ADD users "user:1001"
# Returns: 1 (added) or 0 (might already exist)

# Add multiple items
BF.MADD users "user:1002" "user:1003" "user:1004"
# Returns: [1, 1, 1]
```

#### BF.EXISTS / BF.MEXISTS: Check Membership

```redis
# Check single item
BF.EXISTS users "user:1001"
# Returns: 1 (probably exists) or 0 (definitely not)

# Check multiple items
BF.MEXISTS users "user:1001" "user:9999" "unknown"
# Returns: [1, 0, 0]
```

#### BF.INFO: Filter Statistics

```redis
BF.INFO users
# Returns:
# Capacity: 1000000
# Size: 1198696
# Number of filters: 1
# Number of items inserted: 500000
# Expansion rate: 2
```

### Error Rate vs Memory

| Error Rate | Bits per Item | For 1M items |
|------------|---------------|--------------|
| 10%        | 4.8 bits      | 0.6 MB       |
| 1%         | 9.6 bits      | 1.2 MB       |
| 0.1%       | 14.4 bits     | 1.8 MB       |
| 0.01%      | 19.2 bits     | 2.4 MB       |

## Common Pitfalls

1. **Not reserving capacity upfront** — If you skip BF.RESERVE and just call BF.ADD, Redis auto-creates a filter with default settings (capacity 100, error rate 1%). This is almost never what you want in production.
2. **Exceeding capacity without EXPANSION** — Once a non-expandable filter exceeds its capacity, the false positive rate degrades rapidly. Always set EXPANSION or monitor item count with BF.INFO.

## Best Practices

1. **Always use BF.RESERVE** — Explicitly set error rate and capacity based on your use case. A 1% error rate with 2x your expected items is a solid starting point.
2. **Use BF.EXISTS as a pre-filter** — Check the Bloom filter first to avoid expensive database lookups. If the filter says "no", skip the lookup entirely.

## Summary

- Bloom filters answer membership queries with zero false negatives and a configurable false positive rate
- BF.RESERVE creates a filter; BF.ADD inserts items; BF.EXISTS checks membership
- Memory usage depends on error rate and capacity, not on the actual data stored
- Use Bloom filters as a fast pre-filter layer before hitting slower storage

## Code Examples

**Event deduplication using a Bloom filter as a fast pre-check**

```python
import redis

r = redis.Redis()

def setup_dedup_filter(name, error_rate=0.01, capacity=1000000):
    """Create a deduplication filter"""
    try:
        r.execute_command('BF.RESERVE', name, error_rate, capacity)
    except redis.ResponseError:
        pass  # Filter already exists

def is_duplicate(filter_name, item):
    """Check if item is a duplicate (might be false positive)"""
    return r.execute_command('BF.EXISTS', filter_name, item) == 1

def mark_seen(filter_name, item):
    """Mark item as seen"""
    return r.execute_command('BF.ADD', filter_name, item)

def process_event(event_id):
    """Process event only if not seen before"""
    if is_duplicate('events:seen', event_id):
        return False  # Skip duplicate
    
    # Process the event...
    mark_seen('events:seen', event_id)
    return True
```


## Resources

- [Bloom Filters](https://redis.io/docs/latest/develop/data-types/probabilistic/bloom-filter/) — Redis Bloom Filter documentation
- [BF.RESERVE Command](https://redis.io/docs/latest/commands/bf.reserve/) — BF.RESERVE command reference

---

> 📘 *This lesson is part of the [Probabilistic Data Structures and Analytics](https://stanza.dev/courses/redis-probabilistic) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*