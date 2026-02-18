---
source_course: "redis-probabilistic"
source_lesson: "redis-probabilistic-intro"
---

# Why Probabilistic Data Structures?

Probabilistic data structures trade perfect accuracy for dramatic memory and speed improvements. They're essential for big data scenarios.

## The Scale Problem

```
┌─────────────────────────────────────────────────────────────┐
│  Problem: Count unique visitors to your website             │
│                                                             │
│  Exact Solution: Store every visitor ID                    │
│  - 1 billion visitors × 36 bytes (UUID) = 36 GB            │
│                                                             │
│  Probabilistic Solution: HyperLogLog                       │
│  - 1 billion visitors = 12 KB (!)                          │
│  - 99.19% accurate (error < 1%)                            │
└─────────────────────────────────────────────────────────────┘
```

## Common Questions

### "How many unique users visited today?"

**Exact**: Store all user IDs → Check size
**Probabilistic**: HyperLogLog → 12 KB, ~0.81% error

### "Has this user seen this ad before?"

**Exact**: Store all user-ad pairs → Check existence
**Probabilistic**: Bloom Filter → Small memory, may have false positives

### "What are the top 10 trending topics?"

**Exact**: Count all topic occurrences → Sort
**Probabilistic**: Top-K → Fixed memory, approximate ranking

## Accuracy vs Resources Trade-off

```
┌─────────────────────────────────────────────────────────────┐
│  Accuracy                                                   │
│  100% │                                                     │
│       │     ────────────  Exact (unbounded memory)         │
│  99%  │  ──────────        Probabilistic                   │
│       │                    (fixed memory)                  │
│       └─────────────────────────────────────────────────────│
│                                      Memory Usage          │
└─────────────────────────────────────────────────────────────┘
```

## Redis Probabilistic Structures

| Structure | Use Case | Error Type |
|-----------|----------|------------|
| HyperLogLog | Count unique items | ±0.81% on count |
| Bloom Filter | Check if item exists | False positives possible |
| Cuckoo Filter | Bloom + deletion support | False positives possible |
| Count-Min Sketch | Estimate item frequency | Overestimates possible |
| Top-K | Find most frequent items | Approximate ranking |
| t-digest | Percentile estimation | Approximate percentiles |

## When to Use Probabilistic Structures

✅ **Good candidates:**
- Analytics and counting (unique visitors, events)
- Deduplication at scale
- Rate limiting checks
- Recommendation filtering
- Spam detection

❌ **Avoid when:**
- Exact counts are legally required
- False positives cause serious problems
- Dataset is small (exact is fine)
- Individual item lookup is needed

## Memory Comparison

| Task | Exact | Probabilistic |
|------|-------|---------------|
| 1B unique counts | 36 GB | 12 KB (HyperLogLog) |
| 1B membership tests | 36 GB | 1.2 GB (Bloom Filter) |
| Top 100 of 1B items | 36 GB | <1 MB (Top-K) |

📖 [Probabilistic Data Types](https://redis.io/docs/latest/develop/data-types/probabilistic/)

## Resources

- [Probabilistic Data Types](https://redis.io/docs/latest/develop/data-types/probabilistic/) — Overview of Redis probabilistic data structures

---

> 📘 *This lesson is part of the [Probabilistic Data Structures and Analytics](https://stanza.dev/courses/redis-probabilistic) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*