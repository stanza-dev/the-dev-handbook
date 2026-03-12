---
source_course: "redis-probabilistic"
source_lesson: "redis-probabilistic-error-types"
---

# Types of Probabilistic Errors

## Introduction

Not all probabilistic errors are created equal. A Bloom filter that says "yes" might be wrong, but one that says "no" is always right. Understanding these error types is essential for choosing the right structure and designing systems that handle uncertainty correctly.

## Key Concepts

- **False positive**: The structure says an item exists (or has a high count) when it actually does not. Bloom filters and Cuckoo filters can produce these.
- **False negative**: The structure says an item does not exist when it actually does. Standard Bloom filters never produce these.
- **One-sided error**: An error bound that only goes in one direction — either overestimating or underestimating, but not both.
- **Standard error**: A statistical measure of expected deviation from the true value, expressed as a percentage (e.g., HyperLogLog's 0.81%).

## Real World Context

Consider an ad platform that uses a Bloom filter to avoid showing the same ad twice. A false positive means a user misses seeing an ad they have not actually seen — a minor loss of revenue. A false negative would mean showing the same ad repeatedly — annoying the user. Because Bloom filters guarantee no false negatives, the annoying case never happens. You accept the rare missed impression in exchange for massive memory savings.

## Deep Dive

Each Redis probabilistic structure has a specific error profile:

### Bloom Filter and Cuckoo Filter: False Positives Only

These structures answer "is X in the set?" and can only err on the positive side:

- If the filter says **no** — the item is definitely not in the set.
- If the filter says **yes** — the item is probably in the set, but might not be.

The false positive rate is configurable when you create the filter:

```redis
# 1% false positive rate, capacity 1 million items
BF.RESERVE myfilter 0.01 1000000
```

A lower error rate requires more memory (more bits per item). The relationship is roughly:

| Error Rate | Bits per Item |
|------------|---------------|
| 10% | ~4.8 bits |
| 1% | ~9.6 bits |
| 0.1% | ~14.4 bits |
| 0.01% | ~19.2 bits |

### HyperLogLog: Symmetric Error on Count

HyperLogLog estimates cardinality with a standard error of 0.81%. This error is symmetric — it can overestimate or underestimate the true count, but the magnitude is bounded.

For a true count of 1,000,000, PFCOUNT will typically return a value between 991,900 and 1,008,100.

### Count-Min Sketch: Overestimates Only

Count-Min Sketch estimates how often an item appears. Due to hash collisions, counters can only be incremented by colliding items — never decremented. The result is always greater than or equal to the true count.

```redis
# Error margin 0.1%, 99% confidence
CMS.INITBYPROB mycms 0.001 0.01
```

### Top-K: Approximate Ranking

Top-K maintains an approximate list of the most frequent items. Items may enter or leave the list incorrectly at boundary frequencies. An item ranked 11th might appear in a top-10 list while the actual 10th item is excluded.

### t-digest: Percentile Error

t-digest estimates percentiles (median, p95, p99). The error depends on the compression parameter and is generally smaller at the extremes (p1, p99) and larger near the median. This makes it especially good for SLA monitoring where tail latencies matter most.

## Common Pitfalls

1. **Confusing false positive rate with total error** — A 1% false positive rate does not mean 1% of your data is wrong. It means that for items NOT in the set, 1% of checks will incorrectly say "yes." For items that ARE in the set, the answer is always correct.
2. **Forgetting that Bloom filter error grows with fill ratio** — As a Bloom filter fills beyond its reserved capacity, the false positive rate increases beyond the configured rate. Always size your filter for expected capacity with headroom.
3. **Assuming Count-Min Sketch errors are symmetric** — CMS only overestimates. If you see a count of 50, the true count could be anywhere from 1 to 50, but never more than 50. Design your logic accordingly.

## Best Practices

1. **Match the error type to your tolerance** — If false negatives are catastrophic (security checks, deduplication), use Bloom filters that guarantee no false negatives. If overcounting is acceptable but undercounting is not, Count-Min Sketch is safe.
2. **Size structures for peak, not average** — Reserve capacity for your expected peak data volume with 2-3x headroom. Under-sizing degrades accuracy, sometimes dramatically.
3. **Log and monitor actual error rates** — Periodically compare probabilistic answers against exact samples to verify your error rates match expectations in production.

## Summary

- False positives ("yes" when the answer is "no") affect Bloom filters and Cuckoo filters.
- False negatives ("no" when the answer is "yes") do not occur in standard Bloom filters.
- HyperLogLog has symmetric error (can be above or below the true count) bounded at 0.81% standard error.
- Count-Min Sketch only overestimates — it never reports a count lower than the true value.
- Understanding error direction lets you design systems that fail safely.

## Code Examples

**Demonstrating that Bloom filters produce false positives but never false negatives**

```python
import redis

r = redis.Redis()

# Demonstrate false positive in Bloom filter
r.execute_command('BF.RESERVE', 'demo:bloom', 0.1, 100)  # 10% FP rate

# Add some items
for i in range(100):
    r.execute_command('BF.ADD', 'demo:bloom', f'item:{i}')

# Check items that were NEVER added
false_positives = 0
tests = 1000
for i in range(100, 100 + tests):
    if r.execute_command('BF.EXISTS', 'demo:bloom', f'item:{i}'):
        false_positives += 1

print(f'False positive rate: {false_positives/tests:.1%}')  # ~10%

# Check items that WERE added — always found (no false negatives)
for i in range(100):
    assert r.execute_command('BF.EXISTS', 'demo:bloom', f'item:{i}') == 1
```


## Resources

- [Probabilistic Data Types](https://redis.io/docs/latest/develop/data-types/probabilistic/) — Overview of all Redis probabilistic data structures and their error characteristics

---

> 📘 *This lesson is part of the [Probabilistic Data Structures and Analytics](https://stanza.dev/courses/redis-probabilistic) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*