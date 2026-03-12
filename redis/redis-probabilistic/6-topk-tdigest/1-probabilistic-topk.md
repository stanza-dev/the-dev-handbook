---
source_course: "redis-probabilistic"
source_lesson: "redis-probabilistic-topk"
---

# Top-K: Finding Most Frequent Items

## Introduction

When you have millions of events streaming in, identifying the most popular items in real time is a surprisingly hard problem. Top-K solves it with fixed memory by maintaining an approximate leaderboard that updates with every new event.

## Key Concepts

- **Heavy hitters**: Items that appear disproportionately often in a data stream. Top-K tracks these efficiently.
- **Min-heap**: Internally, Top-K maintains a heap of the K items with the highest estimated counts. When a new item's count surpasses the heap's minimum, it replaces the weakest entry.
- **Decay factor**: A parameter (0 < decay ≤ 1) that controls how quickly old counts fade, enabling recency-biased trending.
- **Displaced items**: When TOPK.ADD causes an item to enter the top K, it returns the item that was kicked out (if any).

## Real World Context

Trending hashtags, popular products, most-visited pages, and heavy-hitter detection in network security all need Top-K. Without it, you would have to count every distinct item and sort them — an O(n log n) operation that grows without bound.

## Deep Dive

### Creating a Top-K Structure

```redis
# Basic: track top 10 items
TOPK.RESERVE trending 10

# With tuning parameters
TOPK.RESERVE trending 10 50 4 0.9
# k=10, width=50, depth=4, decay=0.9
```

The optional parameters (width, depth, decay) tune the internal Count-Min Sketch that Top-K uses for frequency estimation.

### Adding Items

```redis
# Add items (each occurrence increments count by 1)
TOPK.ADD trending "product:1" "product:2" "product:1" "product:3"
# Returns items displaced from the top-K (or nil)

# Increment by specific amount
TOPK.INCRBY trending "product:1" 10 "product:2" 5
```

### Querying the Top-K

```redis
# List all top-K items
TOPK.LIST trending
# Returns: ["product:1", "product:3", "product:2", ...]

# List with approximate counts
TOPK.LIST trending WITHCOUNT
# Returns: ["product:1", "15", "product:2", "6", ...]

# Check if specific items are in the top-K
TOPK.QUERY trending "product:1" "product:999"
# Returns: [1, 0]
```

### How It Works Internally

```
┌─────────────────────────────────────────────────────────────┐
│  1. Uses a Count-Min Sketch for frequency estimates        │
│  2. Maintains a min-heap of top K items                    │
│  3. On each ADD:                                           │
│     - Increment item's count in the CMS                    │
│     - If count > min of heap AND item not in heap:         │
│       → Remove minimum from heap                           │
│       → Add new item to heap                               │
│       → Return displaced item                              │
└─────────────────────────────────────────────────────────────┘
```

### Practical Example: Trending Hashtags

```python
import redis

r = redis.Redis(decode_responses=True)

class TrendingTracker:
    def __init__(self, name, top_k=10):
        self.key = f'trending:{name}'
        try:
            r.execute_command('TOPK.RESERVE', self.key, top_k)
        except redis.ResponseError:
            pass

    def record_activity(self, item, count=1):
        """Record activity for an item."""
        if count == 1:
            displaced = r.execute_command('TOPK.ADD', self.key, item)
            return displaced  # Items kicked out of top-K
        else:
            return r.execute_command('TOPK.INCRBY', self.key, item, count)

    def get_trending(self, with_counts=True):
        """Get trending items."""
        if with_counts:
            result = r.execute_command('TOPK.LIST', self.key, 'WITHCOUNT')
            return {result[i]: int(result[i+1]) for i in range(0, len(result), 2)}
        return r.execute_command('TOPK.LIST', self.key)

    def is_trending(self, item):
        """Check if item is currently trending."""
        result = r.execute_command('TOPK.QUERY', self.key, item)
        return result[0] == 1

trending = TrendingTracker('hashtags', top_k=20)
trending.record_activity('#redis')
trending.record_activity('#python')
trending.record_activity('#redis')
print(trending.get_trending())  # {'#redis': 2, '#python': 1, ...}
```

## Common Pitfalls

1. **Treating Top-K counts as exact** — The counts returned by TOPK.LIST WITHCOUNT are approximations from the internal CMS. They are useful for ranking but not for precise analytics.
2. **Setting K too high** — Top-K is designed for small K values (10-100). For very large K, the overhead approaches that of exact counting, defeating the purpose.

## Best Practices

1. **Use TOPK.ADD return values for eviction notifications** — When an item is displaced, you can log or react to it (e.g., send a "no longer trending" event).
2. **Tune the decay parameter for your use case** — A decay of 0.9 favors recent events; 1.0 treats all events equally regardless of arrival time.

## Summary

- TOPK.RESERVE creates a fixed-memory structure for tracking the K most frequent items in a stream.
- TOPK.ADD and TOPK.INCRBY update counts; TOPK.LIST retrieves the current top-K with optional counts.
- Internally, Top-K combines a Count-Min Sketch with a min-heap for efficient streaming updates.
- Ideal for trending topics, popular products, and heavy-hitter detection.

## Code Examples

**Tracking trending search terms with Top-K, including displaced item detection**

```python
import redis

r = redis.Redis(decode_responses=True)

# Create a Top-K for trending search terms
try:
    r.execute_command('TOPK.RESERVE', 'trending:searches', 10)
except redis.ResponseError:
    pass

# Record search queries as they arrive
queries = ['redis', 'python', 'redis', 'docker', 'redis', 'python', 'kubernetes']
for q in queries:
    displaced = r.execute_command('TOPK.ADD', 'trending:searches', q)
    if displaced and displaced[0] is not None:
        print(f'Displaced from top-K: {displaced[0]}')

# Get the current trending searches with counts
result = r.execute_command('TOPK.LIST', 'trending:searches', 'WITHCOUNT')
trending = {result[i]: int(result[i+1]) for i in range(0, len(result), 2)}
print(trending)  # {'redis': 3, 'python': 2, 'docker': 1, 'kubernetes': 1}
```


## Resources

- [Top-K](https://redis.io/docs/latest/develop/data-types/probabilistic/top-k/) — Redis Top-K documentation
- [Top-K Commands](https://redis.io/docs/latest/commands/?group=topk) — Full reference for all Top-K commands

---

> 📘 *This lesson is part of the [Probabilistic Data Structures and Analytics](https://stanza.dev/courses/redis-probabilistic) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*