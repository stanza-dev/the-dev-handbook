---
source_course: "redis-probabilistic"
source_lesson: "redis-probabilistic-bloom-intro"
---

# Understanding Bloom Filters

A Bloom filter is a space-efficient probabilistic data structure for membership testing. It can tell you if an item "definitely is not" or "possibly is" in a set.

## How It Works

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

## Key Properties

- **No false negatives**: If Bloom says "no", the item is definitely not in the set
- **False positives possible**: If Bloom says "yes", the item might be in the set
- **No deletion**: Can't remove items (use Cuckoo filter if needed)
- **No enumeration**: Can't list what's in the filter

## Basic Commands

### BF.RESERVE: Create Filter

```redis
# Create a filter with error rate and capacity
BF.RESERVE users 0.01 1000000
# 0.01 = 1% false positive rate
# 1000000 = expected number of items

# With expansion (auto-grow)
BF.RESERVE users 0.01 1000000 EXPANSION 2
```

### BF.ADD: Add Items

```redis
# Add single item
BF.ADD users "user:1001"
# Returns: 1 (added) or 0 (might exist)

# Add multiple items
BF.MADD users "user:1002" "user:1003" "user:1004"
# Returns: [1, 1, 1]
```

### BF.EXISTS: Check Membership

```redis
# Check single item
BF.EXISTS users "user:1001"
# Returns: 1 (probably exists) or 0 (definitely not)

# Check multiple items
BF.MEXISTS users "user:1001" "user:9999" "unknown"
# Returns: [1, 0, 0]
```

## Error Rate vs Memory

```
┌─────────────────────────────────────────────────────────────┐
│  Error Rate    │  Bits per Item  │  For 1M items           │
│  ─────────────┼─────────────────┼────────────────────────│
│  10%           │  4.8 bits       │  0.6 MB                 │
│  1%            │  9.6 bits       │  1.2 MB                 │
│  0.1%          │  14.4 bits      │  1.8 MB                 │
│  0.01%         │  19.2 bits      │  2.4 MB                 │
└─────────────────────────────────────────────────────────────┘
```

## Practical Example: Deduplication

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
    
    # Process the event
    process(event_id)
    
    # Mark as seen
    mark_seen('events:seen', event_id)
    return True
```

## Use Cases

1. **Email spam detection**: Has this sender been flagged?
2. **Duplicate URL detection**: Have we crawled this URL?
3. **Username availability**: Quick check before database query
4. **Cache warming**: Is this item likely in cache?
5. **Recommendation filtering**: Don't show already-seen items

## BF.INFO: Filter Statistics

```redis
BF.INFO users
# Returns:
# Capacity: 1000000
# Size: 1198696
# Number of filters: 1
# Number of items inserted: 500000
# Expansion rate: 2
```

📖 [Bloom Filter Commands](https://redis.io/docs/latest/develop/data-types/probabilistic/bloom-filter/)

## Resources

- [Bloom Filters](https://redis.io/docs/latest/develop/data-types/probabilistic/bloom-filter/) — Redis Bloom Filter documentation

---

> 📘 *This lesson is part of the [Probabilistic Data Structures and Analytics](https://stanza.dev/courses/redis-probabilistic) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*