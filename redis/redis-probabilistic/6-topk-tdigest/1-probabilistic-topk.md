---
source_course: "redis-probabilistic"
source_lesson: "redis-probabilistic-topk"
---

# Top-K: Finding Most Frequent Items

Top-K data structure maintains an approximate list of the K most frequent items in a stream.

## The Problem

```
┌─────────────────────────────────────────────────────────────┐
│  Problem: Find the 10 most popular products                │
│                                                             │
│  Exact: Sort all products by count → O(n log n)           │
│  Memory: Store all product counts                          │
│                                                             │
│  Top-K: Fixed memory, streaming updates                    │
│  Memory: O(K) - only store K items                         │
└─────────────────────────────────────────────────────────────┘
```

## Basic Commands

### TOPK.RESERVE: Create Top-K

```redis
# Create Top-K structure for top 10 items
TOPK.RESERVE trending 10

# With tuning parameters
TOPK.RESERVE trending 10 50 4 0.9
# k=10, width=50, depth=4, decay=0.9
```

### TOPK.ADD: Add Items

```redis
# Add items (increment their counts)
TOPK.ADD trending "product:1" "product:2" "product:1" "product:3"
# Returns items that were dropped from top-K (if any)
```

### TOPK.LIST: Get Top Items

```redis
# List top K items
TOPK.LIST trending
# Returns: ["product:1", "product:3", "product:2", ...]

# With counts
TOPK.LIST trending WITHCOUNT
# Returns: ["product:1", "5", "product:3", "3", ...]
```

### TOPK.QUERY: Check if Item is in Top-K

```redis
# Check if items are in the top-K
TOPK.QUERY trending "product:1" "product:999"
# Returns: [1, 0] (product:1 is in top-K, product:999 is not)
```

### TOPK.INCRBY: Increment by Specific Amount

```redis
# Increment by specific amount
TOPK.INCRBY trending "product:1" 10 "product:2" 5
```

## Practical Example: Trending Topics

```python
import redis

r = redis.Redis(decode_responses=True)

class TrendingTracker:
    def __init__(self, name, top_k=10):
        self.key = f'trending:{name}'
        try:
            r.execute_command('TOPK.RESERVE', self.key, top_k)
        except redis.ResponseError:
            pass  # Already exists
    
    def record_activity(self, item, count=1):
        """Record activity for an item"""
        if count == 1:
            r.execute_command('TOPK.ADD', self.key, item)
        else:
            r.execute_command('TOPK.INCRBY', self.key, item, count)
    
    def get_trending(self, with_counts=True):
        """Get trending items"""
        if with_counts:
            result = r.execute_command('TOPK.LIST', self.key, 'WITHCOUNT')
            # Convert to dict
            return {result[i]: int(result[i+1]) for i in range(0, len(result), 2)}
        else:
            return r.execute_command('TOPK.LIST', self.key)
    
    def is_trending(self, item):
        """Check if item is currently trending"""
        result = r.execute_command('TOPK.QUERY', self.key, item)
        return result[0] == 1

# Usage
trending = TrendingTracker('hashtags', top_k=20)
trending.record_activity('#redis')
trending.record_activity('#python')
trending.record_activity('#redis')  # More popular

print(trending.get_trending())
# {'#redis': 2, '#python': 1, ...}
```

## Use Cases

1. **Trending hashtags/topics**
2. **Popular products**
3. **Frequently searched terms**
4. **Top URLs/pages**
5. **Heavy hitter detection (network)**

## How Top-K Works Internally

1. Uses a Count-Min Sketch for counts
2. Maintains a min-heap of top K items
3. When new item's count exceeds min of heap:
   - Remove minimum from heap
   - Add new item to heap

📖 [Top-K Commands](https://redis.io/docs/latest/develop/data-types/probabilistic/top-k/)

## Resources

- [Top-K](https://redis.io/docs/latest/develop/data-types/probabilistic/top-k/) — Redis Top-K documentation

---

> 📘 *This lesson is part of the [Probabilistic Data Structures and Analytics](https://stanza.dev/courses/redis-probabilistic) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*