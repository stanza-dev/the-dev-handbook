---
source_course: "redis-probabilistic"
source_lesson: "redis-probabilistic-cms-intro"
---

# Count-Min Sketch Explained

## Introduction

Counting how often things happen sounds trivial until you have billions of events per day. Count-Min Sketch (CMS) solves frequency estimation in constant memory, making it the go-to structure when you need approximate counts at massive scale.

## Key Concepts

- **Frequency estimation**: Approximating how many times a specific item has appeared in a data stream.
- **Counter matrix**: A grid of counters with `d` rows (hash functions) and `w` columns (counter slots) that stores compressed frequency information.
- **Hash collision**: When two different items map to the same counter position, inflating counts. CMS mitigates this by using multiple hash functions and taking the minimum.
- **Overestimation bias**: CMS can only overestimate counts (never underestimate), because collisions add to counters but never subtract.

## Real World Context

Anytime you need to answer "how many times has X happened?" at scale, CMS is your friend. Ad impression counting, network traffic analysis, page view tracking, and query popularity ranking all benefit from CMS. A single CMS can replace millions of individual counters while using only a few kilobytes of memory.

## Deep Dive

CMS works by hashing each item through `d` independent hash functions. Each hash maps the item to one column in its row. On insertion, all `d` positions are incremented. On query, all `d` positions are read and the minimum value is returned.

```
┌─────────────────────────────────────────────────────────────┐
│  Matrix of counters (d rows × w columns):                  │
│                                                             │
│  hash1(item) → row1, col3                                  │
│  hash2(item) → row2, col7                                  │
│  hash3(item) → row3, col2                                  │
│                                                             │
│  ADD: Increment all hashed positions                       │
│  QUERY: Return MINIMUM of hashed positions                 │
│  (Minimum because collisions only increase counts)         │
└─────────────────────────────────────────────────────────────┘
```

Redis provides two ways to create a CMS:

```redis
# By explicit dimensions (width × depth)
CMS.INITBYDIM wordcount 2000 5

# By error rate and probability
CMS.INITBYPROB wordcount 0.001 0.01
# 0.001 = error margin (epsilon)
# 0.01 = probability of exceeding error (delta)
```

Once created, you add and query items:

```redis
# Increment counts
CMS.INCRBY wordcount "apple" 3 "banana" 2 "cherry" 1

# Query estimated counts
CMS.QUERY wordcount "apple" "banana" "cherry"
# Returns: [3, 2, 1] (approximately)
```

You can also merge multiple sketches, which is useful for combining counts across time windows or distributed nodes:

```redis
CMS.MERGE combined 2 sketch1 sketch2
CMS.MERGE combined 2 sketch1 sketch2 WEIGHTS 1 2
```

## Common Pitfalls

1. **Treating CMS counts as exact** — CMS only provides upper-bound estimates. If your application requires exact counts for small datasets, use a hash map instead.
2. **Using too few columns (width)** — A narrow sketch causes more hash collisions, leading to larger overestimates. Always size your sketch based on expected error tolerance.
3. **Forgetting that CMS never underestimates** — If CMS.QUERY returns 0, the item truly has never been seen. But a non-zero result may be inflated by collisions.

## Best Practices

1. **Use CMS.INITBYPROB for most cases** — Specifying error rate and probability is more intuitive than choosing raw dimensions. Let Redis compute optimal width and depth.
2. **Combine with Top-K for trending analysis** — CMS alone cannot tell you which items are most frequent. Pair it with TOPK.RESERVE to maintain a ranked list of heavy hitters.
3. **Merge sketches for time-windowed analysis** — Create per-hour or per-day sketches and merge them for broader time ranges without re-processing raw data.

## Summary

- Count-Min Sketch estimates item frequencies in constant memory using a matrix of counters and multiple hash functions.
- It never underestimates: reported counts are always greater than or equal to the true count.
- CMS.INITBYPROB lets you specify error tolerance directly; CMS.QUERY returns the minimum across hash functions.
- Ideal for page views, network traffic, ad impressions, and any high-volume counting problem.

## Code Examples

**Basic CMS page view counter using CMS.INITBYPROB, CMS.INCRBY, and CMS.QUERY**

```python
import redis

r = redis.Redis()

class PageViewCounter:
    def __init__(self):
        self.key = 'pageviews:counter'
        try:
            r.execute_command('CMS.INITBYPROB', self.key, 0.001, 0.01)
        except redis.ResponseError:
            pass  # Already exists

    def record_view(self, page_url):
        """Record a page view"""
        r.execute_command('CMS.INCRBY', self.key, page_url, 1)

    def get_views(self, page_url):
        """Get approximate view count"""
        result = r.execute_command('CMS.QUERY', self.key, page_url)
        return result[0]

    def get_multiple_views(self, *page_urls):
        """Get view counts for multiple pages"""
        result = r.execute_command('CMS.QUERY', self.key, *page_urls)
        return dict(zip(page_urls, result))

counter = PageViewCounter()
counter.record_view('/home')
counter.record_view('/about')
counter.record_view('/home')
print(counter.get_views('/home'))  # ~2
```


## Resources

- [Count-Min Sketch](https://redis.io/docs/latest/develop/data-types/probabilistic/count-min-sketch/) — Redis Count-Min Sketch documentation
- [CMS Commands](https://redis.io/docs/latest/commands/?group=cms) — Full reference for all CMS commands

---

> 📘 *This lesson is part of the [Probabilistic Data Structures and Analytics](https://stanza.dev/courses/redis-probabilistic) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*