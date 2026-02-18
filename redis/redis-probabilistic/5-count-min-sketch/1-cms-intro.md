---
source_course: "redis-probabilistic"
source_lesson: "redis-probabilistic-cms-intro"
---

# Count-Min Sketch Explained

Count-Min Sketch (CMS) estimates how many times items occur in a stream using fixed memory, regardless of the number of unique items.

## The Problem

```
┌─────────────────────────────────────────────────────────────┐
│  Problem: Count item frequencies in a data stream          │
│                                                             │
│  Stream: apple, banana, apple, cherry, apple, banana       │
│                                                             │
│  Exact: {"apple": 3, "banana": 2, "cherry": 1}             │
│  Memory: O(number of unique items)                         │
│                                                             │
│  CMS: Fixed-size counter matrix                            │
│  Memory: O(1) - constant regardless of items               │
└─────────────────────────────────────────────────────────────┘
```

## How It Works

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

## Key Properties

- **Never underestimates**: Count is always ≥ true count
- **May overestimate**: Due to hash collisions
- **Fixed memory**: Configurable width × depth
- **Fast**: O(depth) for both add and query

## Basic Commands

### CMS.INITBYDIM: Create by Dimensions

```redis
# Create with specific dimensions
CMS.INITBYDIM wordcount 2000 5
# 2000 columns × 5 rows
```

### CMS.INITBYPROB: Create by Error Rate

```redis
# Create with error and probability parameters
CMS.INITBYPROB wordcount 0.001 0.01
# 0.001 = error margin
# 0.01 = probability of exceeding error
```

### CMS.INCRBY: Increment Counts

```redis
# Increment single item
CMS.INCRBY wordcount "apple" 1
# Returns: [count] (new estimated count)

# Increment multiple items
CMS.INCRBY wordcount "apple" 3 "banana" 2 "cherry" 1
# Returns: [apple_count, banana_count, cherry_count]
```

### CMS.QUERY: Get Counts

```redis
# Query single item
CMS.QUERY wordcount "apple"
# Returns: [estimated_count]

# Query multiple items
CMS.QUERY wordcount "apple" "banana" "cherry"
# Returns: [3, 2, 1] (approximately)
```

## Practical Example: Page View Counter

```python
import redis

r = redis.Redis()

class PageViewCounter:
    def __init__(self):
        self.key = 'pageviews:counter'
        try:
            # Error rate 0.1%, 99% confidence
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

# Usage
counter = PageViewCounter()
counter.record_view('/home')
counter.record_view('/about')
counter.record_view('/home')

print(counter.get_views('/home'))  # ~2
print(counter.get_multiple_views('/home', '/about', '/contact'))
```

## CMS.MERGE: Combine Sketches

```redis
# Merge multiple sketches
CMS.MERGE combined 2 sketch1 sketch2

# With weights (for weighted average)
CMS.MERGE combined 2 sketch1 sketch2 WEIGHTS 1 2
```

## Use Cases

1. **Web analytics**: Page view counts
2. **Network monitoring**: Packet frequency by IP
3. **Natural language**: Word frequency in documents
4. **Click tracking**: Ad click counts
5. **Search queries**: Query popularity

📖 [Count-Min Sketch](https://redis.io/docs/latest/develop/data-types/probabilistic/count-min-sketch/)

## Resources

- [Count-Min Sketch](https://redis.io/docs/latest/develop/data-types/probabilistic/count-min-sketch/) — Redis Count-Min Sketch documentation

---

> 📘 *This lesson is part of the [Probabilistic Data Structures and Analytics](https://stanza.dev/courses/redis-probabilistic) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*