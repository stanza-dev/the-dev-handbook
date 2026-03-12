---
source_course: "redis-probabilistic"
source_lesson: "redis-probabilistic-bloom-sizing"
---

# Bloom Filter Sizing and Error Rates

## Introduction

Choosing the right size for a Bloom filter is the difference between a fast, memory-efficient lookup and a filter that either wastes RAM or drowns you in false positives. Understanding BF.RESERVE parameters lets you make that trade-off deliberately.

## Key Concepts

- **Capacity**: The number of items the filter is designed to hold before error rate guarantees degrade
- **Error Rate**: The target false positive probability (e.g., 0.01 means 1% of checks on non-existent items will incorrectly return "exists")
- **Bits per Item**: The number of bits allocated per expected item — directly determines memory usage and accuracy
- **Expansion Factor**: When a sub-filter reaches capacity, the next sub-filter is created with `capacity * expansion` slots
- **Number of Hash Functions**: Automatically calculated from the error rate — lower error rates need more hash functions

## Real World Context

An ad-tech company serving 500 million ad impressions per day needs to deduplicate events. Under-sizing the filter means the false positive rate climbs and legitimate impressions get dropped — costing revenue. Over-sizing wastes expensive RAM on Redis nodes. Getting BF.RESERVE right saves both money and correctness.

## Deep Dive

### BF.RESERVE Parameters

```redis
BF.RESERVE <key> <error_rate> <capacity> [EXPANSION <factor>] [NONSCALING]
```

- `error_rate`: Target false positive rate (0 to 1). Smaller = more memory.
- `capacity`: Expected number of unique items. Set this to your best estimate.
- `EXPANSION <factor>`: Growth factor when capacity is exceeded. Default is 2. Each new sub-filter has `factor * previous_capacity` slots but a tighter error rate.
- `NONSCALING`: Disables expansion. The filter returns an error when full.

### The Math Behind Sizing

For a given error rate `p` and capacity `n`:

- **Bits needed**: `m = -n * ln(p) / (ln(2))^2`
- **Optimal hash functions**: `k = (m / n) * ln(2)`

Example: 1 million items at 1% error rate:
- Bits: `m = -1,000,000 * ln(0.01) / 0.4805 ≈ 9,585,058` → ~1.2 MB
- Hash functions: `k ≈ 7`

### Practical Sizing Table

| Items | Error Rate | Memory   | Hash Functions |
|-------|-----------|----------|----------------|
| 100K  | 1%        | 120 KB   | 7              |
| 1M    | 1%        | 1.2 MB   | 7              |
| 1M    | 0.1%      | 1.8 MB   | 10             |
| 10M   | 1%        | 12 MB    | 7              |
| 10M   | 0.01%     | 24 MB    | 13             |
| 100M  | 1%        | 120 MB   | 7              |

### Expansion Behavior

When a filter reaches capacity, Redis creates a new sub-filter:

```redis
# Create filter expecting 100,000 items, grows 2x when full
BF.RESERVE myfilter 0.01 100000 EXPANSION 2

# After inserting 100,001 items:
# Sub-filter 1: capacity 100,000, error rate ~0.005
# Sub-filter 2: capacity 200,000, error rate ~0.005
# Combined error rate stays ≤ 0.01
```

Each sub-filter gets a tighter individual error rate so the combined rate stays at or below your target. However, each additional sub-filter adds latency because all sub-filters must be checked on BF.EXISTS.

### Monitoring Filter Health

```redis
BF.INFO myfilter
# Capacity: 100000
# Size: 239744         ← memory in bytes
# Number of filters: 1 ← sub-filter count (1 = not yet expanded)
# Number of items inserted: 45230
# Expansion rate: 2
```

Watch `Number of filters` — if it grows beyond 3-4, consider recreating the filter with a larger initial capacity.

## Common Pitfalls

1. **Setting capacity too low** — The filter expands via sub-filters, which works but degrades lookup performance. Each BF.EXISTS must check every sub-filter. A filter with 10 sub-filters is 10x slower than one with 1.
2. **Confusing error rate with miss rate** — The error rate is the probability of a false POSITIVE, not a false negative. False negatives never happen. Setting error_rate to 0.01 means 1 in 100 non-existent items will incorrectly appear to exist.

## Best Practices

1. **Estimate capacity at 2x your expected items** — It is cheaper to allocate extra bits upfront than to pay the performance cost of multiple sub-filters expanding at runtime.
2. **Use BF.INFO in monitoring** — Alert when `Number of items inserted` approaches `Capacity` or when `Number of filters` exceeds 2. This gives you time to migrate to a larger filter before performance degrades.

## Summary

- BF.RESERVE takes an error rate (false positive probability) and a capacity (expected items)
- Memory is determined by the formula `m = -n * ln(p) / (ln(2))^2` — roughly 9.6 bits per item at 1% error
- EXPANSION creates sub-filters when capacity is exceeded, but each sub-filter adds lookup latency
- Monitor with BF.INFO and plan capacity at 2x your estimate to avoid sub-filter sprawl

## Code Examples

**Calculate Bloom filter memory requirements and create a properly sized filter**

```python
import redis
import math

r = redis.Redis()

def calculate_bloom_size(capacity, error_rate):
    """Calculate memory needed for a Bloom filter."""
    bits = -capacity * math.log(error_rate) / (math.log(2) ** 2)
    hash_count = int((bits / capacity) * math.log(2))
    memory_bytes = math.ceil(bits / 8)
    return {
        'bits': int(bits),
        'hash_functions': hash_count,
        'memory_bytes': memory_bytes,
        'memory_mb': round(memory_bytes / (1024 * 1024), 2)
    }

def create_sized_filter(name, expected_items, error_rate=0.01, safety_factor=2):
    """Create a Bloom filter sized with a safety margin."""
    capacity = expected_items * safety_factor
    sizing = calculate_bloom_size(capacity, error_rate)
    print(f"Creating filter: {capacity} capacity, "
          f"{sizing['memory_mb']} MB, "
          f"{sizing['hash_functions']} hash functions")
    try:
        r.execute_command('BF.RESERVE', name, error_rate, capacity, 'EXPANSION', 2)
    except redis.ResponseError:
        pass
    return sizing

# Size for 5 million expected URLs at 0.1% error
sizing = create_sized_filter('crawled_urls', 5_000_000, error_rate=0.001)
# Creating filter: 10000000 capacity, 17.93 MB, 10 hash functions
```


## Resources

- [BF.RESERVE Command](https://redis.io/docs/latest/commands/bf.reserve/) — BF.RESERVE command reference with parameter details
- [Bloom Filters](https://redis.io/docs/latest/develop/data-types/probabilistic/bloom-filter/) — Redis Bloom Filter overview and sizing guidance

---

> 📘 *This lesson is part of the [Probabilistic Data Structures and Analytics](https://stanza.dev/courses/redis-probabilistic) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*