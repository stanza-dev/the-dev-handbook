---
source_course: "redis-probabilistic"
source_lesson: "redis-probabilistic-choosing-structure"
---

# Choosing the Right Data Structure

## Introduction

With six probabilistic structures available in Redis, picking the right one can feel overwhelming. This lesson gives you a clear decision framework so you can confidently match your problem to the right tool in seconds.

## Key Concepts

- **Cardinality**: The count of distinct elements in a set. Use HyperLogLog.
- **Membership**: Whether an element belongs to a set. Use Bloom or Cuckoo filter.
- **Frequency**: How many times an element appears. Use Count-Min Sketch.
- **Ranking**: Which elements appear most often. Use Top-K.
- **Distribution**: Percentile values across a dataset. Use t-digest.

## Real World Context

You are building an e-commerce analytics dashboard. Different widgets need different structures:

- "Unique visitors today" — HyperLogLog
- "Has user seen this product?" — Bloom filter
- "How many times was product X viewed?" — Count-Min Sketch
- "Top 10 trending products" — Top-K
- "P99 checkout latency" — t-digest

Each widget uses a different structure because each asks a fundamentally different question.

## Deep Dive

Use this decision tree when selecting a structure:

```
What question are you asking?
│
├─ "How many unique items?"      → HyperLogLog
│   Memory: 12 KB (always)
│   Error: ±0.81% standard error
│
├─ "Is this item in the set?"
│   ├─ Need deletion?            → Cuckoo Filter
│   └─ No deletion needed?       → Bloom Filter
│   Memory: ~10 bits/item at 1% FP
│   Error: configurable false positive rate
│
├─ "How often does item appear?" → Count-Min Sketch
│   Memory: fixed (width × depth)
│   Error: overestimates only
│
├─ "What are the top K items?"   → Top-K
│   Memory: O(K)
│   Error: approximate boundary ranking
│
└─ "What is the Nth percentile?" → t-digest
    Memory: fixed (compression param)
    Error: smaller at extremes (p1, p99)
```

### Comparison Table

| Question | Structure | Memory | Error Direction | Configurable? |
|----------|-----------|--------|-----------------|---------------|
| Count unique | HyperLogLog | 12 KB fixed | ± symmetric | No (0.81%) |
| Exists? | Bloom Filter | ~10 bits/item | False positives only | Yes (FP rate) |
| Exists? + delete | Cuckoo Filter | ~12 bits/item | False positives only | Yes (capacity) |
| How many times? | Count-Min Sketch | width × depth | Overestimates only | Yes (error, prob) |
| Most frequent? | Top-K | O(K) | Approximate rank | Yes (K, decay) |
| Percentiles? | t-digest | ~compression × 40B | Varies by quantile | Yes (compression) |

### Combining Structures

In practice, you often combine multiple structures:

```python
import redis

r = redis.Redis()

def track_page_view(user_id, page_url):
    """Track a page view using multiple probabilistic structures."""
    # Count unique visitors (HyperLogLog)
    r.pfadd(f'unique:{page_url}', user_id)

    # Track view frequency (Count-Min Sketch)
    r.execute_command('CMS.INCRBY', 'page:views', page_url, 1)

    # Update trending pages (Top-K)
    r.execute_command('TOPK.ADD', 'trending:pages', page_url)

    # Check if user already seen this page (Bloom filter)
    already_seen = r.execute_command('BF.EXISTS', f'seen:{user_id}', page_url)
    if not already_seen:
        r.execute_command('BF.ADD', f'seen:{user_id}', page_url)
        # First-time view logic here
```

### When NOT to Use Probabilistic Structures

Sometimes exact data structures are better:

- **Small datasets** (< 100K items): A Redis Set or Sorted Set works fine and gives exact answers.
- **Legal or financial requirements**: If regulations demand exact counts, use exact structures.
- **Item retrieval needed**: Probabilistic structures cannot retrieve stored items — they only answer yes/no or how many.
- **Individual record tracking**: If you need to know which specific users visited, not just how many, use a database.

## Common Pitfalls

1. **Using Top-K when you need exact frequency** — Top-K tells you which items are most frequent, not their exact counts. If you need precise counts for the top items, use Top-K to identify candidates, then look up exact counts from a database.
2. **Choosing Cuckoo over Bloom by default** — Cuckoo filters use more memory than Bloom filters. Only choose Cuckoo when you actually need deletion support. For write-once data like crawled URLs or processed event IDs, Bloom is more efficient.
3. **Using Count-Min Sketch for rare items** — CMS overestimates, and the relative error is larger for items with low true counts. A true count of 2 reported as 5 is a 150% error. For rare-item detection, consider different approaches.

## Best Practices

1. **Start with the question, not the structure** — Clearly define what you need to know (unique count, membership, frequency, ranking, percentile) before picking a structure.
2. **Prototype with exact structures first** — Build your feature with Redis Sets or Sorted Sets. Once you hit memory limits, switch to the probabilistic equivalent. This gives you a correctness baseline to validate against.
3. **Use Redis pipelines for multi-structure updates** — When updating multiple probabilistic structures per event, batch commands in a pipeline to reduce round trips.

## Summary

- Match your question to the structure: unique count (HLL), membership (Bloom/Cuckoo), frequency (CMS), ranking (Top-K), percentiles (t-digest).
- Choose Cuckoo over Bloom only when deletion is needed.
- Combine multiple structures to answer different questions about the same data stream.
- Start with exact structures for small datasets; switch to probabilistic when memory becomes a constraint.
- Each structure has a well-defined error direction — design your system to tolerate that specific error type.

## Code Examples

**Choosing the right structure based on the question you need to answer**

```python
import redis

r = redis.Redis()

# Decision: "How many unique users visited /pricing today?"
# Answer: HyperLogLog (cardinality estimation)
r.pfadd('unique:pricing:2024-01-15', 'user:101', 'user:102', 'user:103')
count = r.pfcount('unique:pricing:2024-01-15')
print(f'Unique visitors to /pricing: ~{count}')

# Decision: "Has user:101 already been shown the onboarding tooltip?"
# Answer: Bloom Filter (membership testing, no deletion needed)
r.execute_command('BF.ADD', 'onboarding:shown', 'user:101')
shown = r.execute_command('BF.EXISTS', 'onboarding:shown', 'user:101')
print(f'Tooltip shown: {bool(shown)}')

# Decision: "What is the p99 API response time?"
# Answer: t-digest (percentile estimation)
r.execute_command('TDIGEST.CREATE', 'api:latency')
for ms in [42, 55, 48, 120, 51, 67, 43, 210]:
    r.execute_command('TDIGEST.ADD', 'api:latency', ms)
p99 = r.execute_command('TDIGEST.QUANTILE', 'api:latency', 0.99)
print(f'P99 latency: {p99[0]:.1f} ms')
```


## Resources

- [Probabilistic Data Types](https://redis.io/docs/latest/develop/data-types/probabilistic/) — Overview of all Redis probabilistic data structures
- [HyperLogLog Commands](https://redis.io/docs/latest/commands/?group=hyperloglog) — Redis HyperLogLog command reference

---

> 📘 *This lesson is part of the [Probabilistic Data Structures and Analytics](https://stanza.dev/courses/redis-probabilistic) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*