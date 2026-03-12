---
source_course: "redis-probabilistic"
source_lesson: "redis-probabilistic-hll-intro"
---

# Understanding HyperLogLog

## Introduction

HyperLogLog (HLL) is a probabilistic algorithm that counts unique items using only 12 KB of memory, no matter how many items you add. Whether you are tracking 100 users or 1 billion, the memory footprint stays the same. This lesson covers the core commands — PFADD, PFCOUNT, and PFMERGE — and shows you how to use them for real analytics.

## Key Concepts

- **Cardinality**: The number of distinct elements in a set. HyperLogLog estimates this value.
- **PFADD**: Adds one or more elements to a HyperLogLog. Returns 1 if the internal registers changed, 0 otherwise.
- **PFCOUNT**: Returns the approximate cardinality. When called with multiple keys, it returns the union cardinality.
- **PFMERGE**: Merges multiple HyperLogLogs into one, useful for aggregating time-windowed data.
- **Standard error**: HyperLogLog's error rate is 0.81%, meaning for a true count of 1,000,000, the estimate will typically fall between 991,900 and 1,008,100.

## Real World Context

Every analytics platform needs to answer questions like "how many unique users visited this page today?" or "how many distinct search queries did we receive this week?" Storing every user ID or query string to count them exactly becomes prohibitively expensive at scale. HyperLogLog lets you answer these questions with 12 KB per counter, making it feasible to maintain thousands of counters (one per page, per day, per feature) without worrying about memory.

## Deep Dive

### Adding Items with PFADD

PFADD adds elements to a HyperLogLog and returns whether the cardinality estimate changed:

```redis
# Add a single item
PFADD visitors "user:1001"
# Returns: 1 (cardinality estimate changed)

# Add multiple items at once
PFADD visitors "user:1002" "user:1003" "user:1004"
# Returns: 1

# Adding a duplicate does not change the estimate
PFADD visitors "user:1001"
# Returns: 0 (cardinality unchanged)
```

The return value is useful for understanding whether new unique items were encountered, but it is not a guarantee of uniqueness — it reflects whether internal registers were modified.

### Counting with PFCOUNT

PFCOUNT returns the approximate cardinality of one or more HyperLogLogs:

```redis
# Count unique items in a single HLL
PFCOUNT visitors
# Returns: 4 (approximately)

# Count the union across multiple HLLs
PFCOUNT visitors:day1 visitors:day2 visitors:day3
# Returns: unique items across all three days
```

When called with multiple keys, PFCOUNT computes the union — a user who visited on both day 1 and day 2 is counted only once.

### Merging with PFMERGE

PFMERGE combines multiple HyperLogLogs into a destination key:

```redis
# Merge daily HLLs into a weekly aggregate
PFMERGE visitors:week visitors:mon visitors:tue visitors:wed

# The destination now contains the union
PFCOUNT visitors:week
```

This is essential for building roll-up aggregations — merge daily HLLs into weekly, weekly into monthly.

### Memory Usage

A HyperLogLog always uses approximately 12 KB, regardless of how many items have been added:

```redis
# Check memory usage
MEMORY USAGE visitors
# Returns: approximately 12,304 bytes
```

Redis uses a sparse encoding for small HyperLogLogs (fewer than a threshold of registers set), which uses even less memory. Once enough distinct items are added, it automatically converts to the dense 12 KB representation.

## Common Pitfalls

1. **Using PFCOUNT with many keys in hot paths** — When PFCOUNT is called with multiple keys, Redis computes the union on the fly. For large numbers of keys, this takes time. Use PFMERGE to pre-compute aggregates instead of calling PFCOUNT with dozens of keys on every request.
2. **Expecting PFADD return value to detect duplicates** — PFADD returns 0 when registers do not change, but this does not reliably indicate a duplicate. Due to the probabilistic nature, a genuinely new item might not change any register, and PFADD would return 0.
3. **Forgetting that HLLs cannot be subtracted** — You cannot remove an item from a HyperLogLog or compute set difference. If you need "visitors on day 1 but not day 2," HLL cannot help you directly.

## Best Practices

1. **Use time-bucketed keys** — Create one HLL per time period (e.g., `visitors:2024-01-15`) and merge them for aggregates. This lets you expire old data and keep long-term roll-ups.
2. **Set TTLs on granular keys** — Daily HLLs can be expired after merging into weekly/monthly aggregates, keeping memory usage predictable.
3. **Use PFMERGE for dashboards** — Pre-merge HLLs during off-peak hours rather than computing unions on every dashboard load.

## Summary

- HyperLogLog counts unique items with 0.81% standard error using only 12 KB of memory.
- PFADD adds items, PFCOUNT estimates cardinality, PFMERGE combines multiple HLLs.
- Multi-key PFCOUNT computes the union cardinality (no double-counting).
- Use time-bucketed keys with PFMERGE for daily/weekly/monthly roll-ups.
- HLLs cannot remove items or compute set differences.

## Code Examples

**Daily visitor tracking with weekly roll-up aggregation using PFMERGE**

```python
import redis
from datetime import datetime, timedelta

r = redis.Redis()

def track_visitor(user_id):
    """Track visitor for today."""
    today = datetime.now().strftime('%Y-%m-%d')
    r.pfadd(f'visitors:{today}', user_id)

def get_daily_uniques(date):
    """Get unique visitors for a specific day."""
    return r.pfcount(f'visitors:{date}')

def get_weekly_uniques():
    """Get unique visitors for last 7 days (union)."""
    keys = []
    for i in range(7):
        date = (datetime.now() - timedelta(days=i)).strftime('%Y-%m-%d')
        keys.append(f'visitors:{date}')
    return r.pfcount(*keys)

def create_weekly_rollup():
    """Merge daily HLLs into weekly for long-term storage."""
    keys = []
    for i in range(7):
        date = (datetime.now() - timedelta(days=i)).strftime('%Y-%m-%d')
        keys.append(f'visitors:{date}')
    week = datetime.now().strftime('%Y-W%W')
    r.pfmerge(f'visitors:week:{week}', *keys)
```


## Resources

- [HyperLogLog](https://redis.io/docs/latest/develop/data-types/probabilistic/hyperloglogs/) — Redis HyperLogLog documentation
- [HyperLogLog Commands](https://redis.io/docs/latest/commands/?group=hyperloglog) — PFADD, PFCOUNT, and PFMERGE command reference

---

> 📘 *This lesson is part of the [Probabilistic Data Structures and Analytics](https://stanza.dev/courses/redis-probabilistic) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*