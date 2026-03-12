---
source_course: "redis-probabilistic"
source_lesson: "redis-probabilistic-cuckoo-sizing"
---

# Cuckoo Filter Internals and Sizing

## Introduction

Understanding how Cuckoo filters store data internally — buckets, fingerprints, and kick chains — lets you size them correctly for your workload. Getting BUCKETSIZE and MAXITERATIONS wrong can mean wasted memory or insertion failures under load.

## Key Concepts

- **Bucket**: A fixed-size array that holds multiple fingerprints. Larger buckets improve space efficiency but increase false positive rate slightly.
- **Fingerprint**: A small hash (typically 1-4 bytes) stored in place of the original item. Longer fingerprints reduce false positive rates.
- **Kick / Relocation**: When both candidate buckets are full, an existing fingerprint is evicted to its alternate bucket. This may cascade.
- **MAXITERATIONS**: The maximum number of kick operations before CF.ADD gives up and returns an error. Default is 20.
- **BUCKETSIZE**: Number of fingerprint slots per bucket. Default is 2. Higher values (4, 8) improve load factor but use more memory per bucket.
- **Load Factor**: The ratio of stored items to total capacity. Cuckoo filters work best below 95% load.

## Real World Context

A real-time bidding platform processes 2 million bid requests per second and uses a Cuckoo filter to track active auction IDs. Auctions last 100ms, so items churn rapidly. If MAXITERATIONS is too low, CF.ADD fails during traffic spikes and bids are dropped. If BUCKETSIZE is too small, the filter fills up too quickly. Proper sizing prevents both lost bids and wasted memory.

## Deep Dive

### CF.RESERVE Parameters

```redis
CF.RESERVE <key> <capacity> [BUCKETSIZE <bucketsize>] [MAXITERATIONS <maxiterations>] [EXPANSION <expansion>]
```

- `capacity`: Total number of items the filter can hold. Redis rounds up to the nearest power of 2.
- `BUCKETSIZE`: Fingerprints per bucket (default 2). Allowed values: 1, 2, 4, 8.
- `MAXITERATIONS`: Max kicks before insertion fails (default 20). Higher values allow more items but slow down worst-case inserts.
- `EXPANSION`: Growth factor when the filter is full. Default is 1 (creates a same-size sub-filter).

### Bucket Size Trade-offs

| BUCKETSIZE | Load Factor | False Positive Rate | Memory per Bucket |
|------------|------------|--------------------|---------|
| 1          | ~50%       | Lowest             | Smallest |
| 2          | ~84%       | Low                | Medium   |
| 4          | ~95%       | Medium             | Larger   |
| 8          | ~98%       | Higher             | Largest  |

Higher bucket sizes pack more items into the same number of buckets (better load factor) but each bucket takes more memory and fingerprint collisions within a bucket increase the false positive rate.

### Memory Estimation

A Cuckoo filter's memory usage is approximately:

```
memory = (capacity / load_factor) * fingerprint_size_bits / 8
```

For 1 million items with BUCKETSIZE 2 (84% load factor) and 8-bit fingerprints:
```
memory = (1,000,000 / 0.84) * 1 byte ≈ 1.19 MB
```

### Monitoring with CF.INFO

```redis
CF.INFO myfilter
# Size: 1048576          ← memory in bytes
# Number of buckets: 524288
# Number of filters: 1   ← sub-filter count
# Number of items inserted: 420000
# Number of items deleted: 15000
# Bucket size: 2
# Expansion rate: 1
# Max iterations: 20
```

Key metrics to watch:
- `Number of items inserted` minus `Number of items deleted` = active items
- `Number of filters` > 2 means the filter has expanded and lookups are slower

### Practical Sizing Examples

```redis
# High-throughput session tracking (1M sessions, frequent deletion)
CF.RESERVE sessions 2000000 BUCKETSIZE 4 MAXITERATIONS 50 EXPANSION 2

# Low-churn feature flags (100K users, rare changes)
CF.RESERVE feature_flags 200000 BUCKETSIZE 2 MAXITERATIONS 20

# Real-time dedup with tight memory (10M items, minimal memory)
CF.RESERVE dedup 10000000 BUCKETSIZE 2 MAXITERATIONS 30 EXPANSION 1
```

### Handling Insertion Failures

When the filter is too full, CF.ADD fails:

```python
def safe_add(filter_key, item, fallback_key=None):
    """Add to Cuckoo filter with fallback on failure."""
    try:
        result = r.execute_command('CF.ADD', filter_key, item)
        return result == 1
    except redis.ResponseError as e:
        if 'Maximum expansions reached' in str(e):
            # Filter is full — log and optionally use fallback
            if fallback_key:
                r.sadd(fallback_key, item)  # Exact set as fallback
            return False
        raise
```

## Common Pitfalls

1. **Using default MAXITERATIONS for high-load filters** — The default of 20 is fine for filters below 80% capacity, but at 90%+ load, kick chains get longer. Increase to 50-100 for high-utilization filters to avoid insertion failures.
2. **Not accounting for Redis capacity rounding** — CF.RESERVE rounds capacity up to the nearest power of 2. Requesting 1,000,000 allocates 1,048,576 slots. Plan for this when estimating memory.

## Best Practices

1. **Size capacity at 2x expected active items** — Cuckoo filters perform best below 85% load. Doubling your estimate keeps you in the safe zone and leaves room for traffic spikes.
2. **Use BUCKETSIZE 4 for high-churn workloads** — The 95% load factor of BUCKETSIZE 4 means fewer expansions and more headroom for items being rapidly added and deleted.

## Summary

- CF.RESERVE takes capacity, BUCKETSIZE, MAXITERATIONS, and EXPANSION as parameters
- BUCKETSIZE controls the trade-off between load factor and false positive rate (2 is the default, 4 is good for high churn)
- MAXITERATIONS determines how hard the filter tries before failing an insert (increase for high-load filters)
- Monitor with CF.INFO: watch active item count (inserted minus deleted) and sub-filter count
- Size at 2x expected items to stay below 85% load factor

## Code Examples

**Create and monitor Cuckoo filters sized for different workload patterns**

```python
import redis

r = redis.Redis()

def create_cuckoo_filter(name, expected_items, churn='low'):
    """Create a Cuckoo filter sized for the workload."""
    capacity = expected_items * 2  # 2x safety margin
    
    if churn == 'high':
        bucket_size = 4
        max_iter = 50
        expansion = 2
    else:
        bucket_size = 2
        max_iter = 20
        expansion = 1
    
    try:
        r.execute_command(
            'CF.RESERVE', name, capacity,
            'BUCKETSIZE', bucket_size,
            'MAXITERATIONS', max_iter,
            'EXPANSION', expansion
        )
        print(f"Created '{name}': capacity={capacity}, "
              f"bucket_size={bucket_size}, max_iter={max_iter}")
    except redis.ResponseError:
        pass  # Already exists

def get_filter_health(name):
    """Check Cuckoo filter utilization."""
    info = r.execute_command('CF.INFO', name)
    # Parse info into dict (format varies by client)
    inserted = info[info.index(b'Number of items inserted') + 1]
    deleted = info[info.index(b'Number of items deleted') + 1]
    active = inserted - deleted
    capacity = info[info.index(b'Size') + 1]
    return {'active_items': active, 'memory_bytes': capacity}

# High-churn session tracking
create_cuckoo_filter('sessions', 500_000, churn='high')
# Low-churn feature flags
create_cuckoo_filter('feature:dark_mode', 50_000, churn='low')
```


## Resources

- [CF.RESERVE Command](https://redis.io/docs/latest/commands/cf.reserve/) — CF.RESERVE command reference with BUCKETSIZE and MAXITERATIONS details
- [Cuckoo Filters](https://redis.io/docs/latest/develop/data-types/probabilistic/cuckoo-filter/) — Redis Cuckoo Filter overview and internals

---

> 📘 *This lesson is part of the [Probabilistic Data Structures and Analytics](https://stanza.dev/courses/redis-probabilistic) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*