---
source_course: "redis-probabilistic"
source_lesson: "redis-probabilistic-cms-sizing"
---

# CMS Sizing and Accuracy

## Introduction

Choosing the right dimensions for a Count-Min Sketch determines the balance between memory usage and estimation accuracy. Get it wrong and you either waste memory or get useless estimates. This lesson covers both initialization methods and how to reason about parameter choices.

## Key Concepts

- **Width (w)**: The number of columns in the counter matrix. Larger width means fewer collisions and better accuracy. Computed as `ceil(e / epsilon)` where epsilon is the error rate.
- **Depth (d)**: The number of rows (hash functions). More depth reduces the probability of all hash functions colliding simultaneously. Computed as `ceil(ln(1 / delta))` where delta is the failure probability.
- **Epsilon (error rate)**: The maximum overestimation as a fraction of total count. An epsilon of 0.001 means overestimates stay within 0.1% of the total number of increments.
- **Delta (failure probability)**: The probability that the actual error exceeds epsilon. A delta of 0.01 means 99% of queries will be within the epsilon bound.

## Real World Context

In production, you rarely know the perfect parameters upfront. Understanding how width and depth map to accuracy lets you make informed trade-offs. A real-time bidding platform tracking billions of ad impressions needs tighter error bounds than a dev blog counting page views.

## Deep Dive

Redis offers two ways to create a CMS, and they are mathematically equivalent:

### CMS.INITBYDIM — Direct Control

```redis
# Specify width (columns) and depth (rows) directly
CMS.INITBYDIM traffic 2000 5
# 2000 columns × 5 hash functions
# Memory ≈ 2000 × 5 × 4 bytes = ~40 KB
```

Use INITBYDIM when you have a fixed memory budget and want to control exactly how many counters are allocated.

### CMS.INITBYPROB — Specify Accuracy Goals

```redis
# Specify error rate and failure probability
CMS.INITBYPROB traffic 0.001 0.01
# epsilon = 0.001 → width = ceil(e / 0.001) = 2719
# delta = 0.01 → depth = ceil(ln(1 / 0.01)) = 5
```

Use INITBYPROB when you know your accuracy requirements but not the exact dimensions needed.

### How Parameters Affect Accuracy

```
┌────────────────────────────────────────────────────┐
│  Width ↑  → Fewer collisions → Better accuracy    │
│  Depth ↑  → Lower failure probability             │
│  Both  ↑  → More memory                           │
│                                                    │
│  Example sizing for 10M events/day:                │
│                                                    │
│  Epsilon  │  Width  │  Depth │  Memory   │ MaxErr  │
│  0.01     │  272    │  5     │  ~5 KB    │ 100K    │
│  0.001    │  2719   │  5     │  ~54 KB   │ 10K     │
│  0.0001   │  27183  │  5     │  ~544 KB  │ 1K      │
└────────────────────────────────────────────────────┘
```

The MaxErr column shows the worst-case overestimate: `epsilon × N` where N is the total number of increments. With epsilon = 0.001 and 10 million events, the maximum overestimate on any single item is 10,000.

### Checking Your Sketch Info

```redis
CMS.INFO traffic
# Returns:
# width: 2719
# depth: 5
# count: 0
```

### Sizing Decision Tree

1. **Know your accuracy needs?** Use CMS.INITBYPROB.
2. **Have a memory budget?** Use CMS.INITBYDIM.
3. **Not sure?** Start with `CMS.INITBYPROB 0.001 0.01` — it gives 0.1% error with 99% confidence and uses under 60 KB.

## Common Pitfalls

1. **Confusing epsilon with percentage** — Epsilon 0.01 means 1% of the total count N, not 1% of the individual item's count. A rare item might have a 100% relative error even when the absolute error is small.
2. **Over-sizing depth for diminishing returns** — Going from depth 5 to depth 10 only changes delta from 0.01 to 0.00005. The memory doubles, but the practical improvement is usually not noticeable.

## Best Practices

1. **Start with INITBYPROB and check CMS.INFO** — Let Redis compute dimensions, then verify memory usage with CMS.INFO before deploying to production.
2. **Scale width with expected total count** — If your stream grows 10x, the absolute error grows 10x too. Re-create with tighter epsilon if needed.
3. **Use separate sketches per time window** — Instead of one massive sketch, create hourly or daily sketches. This limits N and keeps absolute error bounded.

## Summary

- CMS.INITBYDIM gives direct control over width (columns) and depth (rows).
- CMS.INITBYPROB lets you specify error rate (epsilon) and failure probability (delta), and Redis computes the dimensions.
- Width controls accuracy; depth controls confidence. Both increase memory.
- For most use cases, `CMS.INITBYPROB 0.001 0.01` is a sensible starting point (~54 KB, 0.1% error, 99% confidence).

## Code Examples

**Comparing CMS.INITBYPROB and CMS.INITBYDIM initialization strategies with memory estimates**

```python
import redis

r = redis.Redis()

def create_cms_by_accuracy(name, error_rate=0.001, probability=0.01):
    """Create a CMS with desired accuracy guarantees."""
    try:
        r.execute_command('CMS.INITBYPROB', name, error_rate, probability)
    except redis.ResponseError:
        pass  # Already exists
    info = r.execute_command('CMS.INFO', name)
    print(f"Created CMS '{name}': width={info[1]}, depth={info[3]}, count={info[5]}")

def create_cms_by_dimensions(name, width, depth):
    """Create a CMS with explicit dimensions for a fixed memory budget."""
    try:
        r.execute_command('CMS.INITBYDIM', name, width, depth)
    except redis.ResponseError:
        pass
    memory = width * depth * 4  # ~4 bytes per counter
    print(f"Created CMS '{name}': ~{memory / 1024:.1f} KB")

# Approach 1: Specify accuracy goals
create_cms_by_accuracy('clicks:accurate', 0.001, 0.01)

# Approach 2: Fixed memory budget
create_cms_by_dimensions('clicks:budget', 1000, 4)
```


## Resources

- [Count-Min Sketch](https://redis.io/docs/latest/develop/data-types/probabilistic/count-min-sketch/) — Redis Count-Min Sketch documentation
- [CMS.INITBYPROB Command](https://redis.io/docs/latest/commands/cms.initbyprob/) — CMS.INITBYPROB command reference

---

> 📘 *This lesson is part of the [Probabilistic Data Structures and Analytics](https://stanza.dev/courses/redis-probabilistic) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*