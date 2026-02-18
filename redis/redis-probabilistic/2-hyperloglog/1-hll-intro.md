---
source_course: "redis-probabilistic"
source_lesson: "redis-probabilistic-hll-intro"
---

# Understanding HyperLogLog

HyperLogLog (HLL) is a probabilistic algorithm for counting unique items (cardinality estimation) using only 12 KB of memory regardless of how many items you add.

## How It Works (Simplified)

```
┌─────────────────────────────────────────────────────────────┐
│  The idea: Hash items and look at leading zeros            │
│                                                             │
│  Item "Alice" → hash → 0001...  (3 leading zeros)          │
│  Item "Bob"   → hash → 0000001... (6 leading zeros)        │
│                                                             │
│  The more unique items you add, the more likely you are    │
│  to see hashes with many leading zeros.                    │
│                                                             │
│  Max leading zeros observed ≈ log2(cardinality)            │
└─────────────────────────────────────────────────────────────┘
```

Redis uses 16,384 registers for better accuracy.

## Basic Commands

### PFADD: Add Items

```redis
# Add single item
PFADD visitors "user:1001"
# Returns: 1 (cardinality changed)

# Add multiple items
PFADD visitors "user:1002" "user:1003" "user:1004"
# Returns: 1

# Adding duplicate
PFADD visitors "user:1001"
# Returns: 0 (cardinality unchanged)
```

### PFCOUNT: Get Cardinality

```redis
# Count unique items
PFCOUNT visitors
# Returns: 4 (approximately)

# Count across multiple HLLs
PFCOUNT visitors:day1 visitors:day2 visitors:day3
# Returns: union cardinality
```

### PFMERGE: Combine HLLs

```redis
# Merge multiple HLLs into one
PFMERGE visitors:week visitors:mon visitors:tue visitors:wed

# The destination contains union of all items
PFCOUNT visitors:week
```

## Accuracy

```
┌─────────────────────────────────────────────────────────────┐
│  Standard Error: 0.81%                                      │
│                                                             │
│  Example: True count = 1,000,000                           │
│  PFCOUNT returns: 991,900 to 1,008,100 (most of the time) │
│                                                             │
│  Memory: 12 KB (always!)                                   │
│  Exact would need: 1M × 36 bytes = 36 MB                   │
└─────────────────────────────────────────────────────────────┘
```

## Practical Example: Daily/Weekly Unique Visitors

```python
import redis
from datetime import datetime, timedelta

r = redis.Redis()

def track_visitor(user_id):
    """Track visitor for today"""
    today = datetime.now().strftime('%Y-%m-%d')
    r.pfadd(f'visitors:{today}', user_id)

def get_daily_uniques(date):
    """Get unique visitors for a specific day"""
    return r.pfcount(f'visitors:{date}')

def get_weekly_uniques():
    """Get unique visitors for last 7 days"""
    keys = []
    for i in range(7):
        date = (datetime.now() - timedelta(days=i)).strftime('%Y-%m-%d')
        keys.append(f'visitors:{date}')
    return r.pfcount(*keys)  # Union of all days

def create_weekly_hll():
    """Merge daily HLLs into weekly for storage"""
    keys = []
    for i in range(7):
        date = (datetime.now() - timedelta(days=i)).strftime('%Y-%m-%d')
        keys.append(f'visitors:{date}')
    
    week = datetime.now().strftime('%Y-W%W')
    r.pfmerge(f'visitors:week:{week}', *keys)
```

## Use Cases

1. **Unique visitors per page/day/month**
2. **Unique search queries**
3. **Unique IP addresses**
4. **Unique products viewed**
5. **Unique events/actions**

## Memory Usage

```redis
# Check memory of a HyperLogLog
MEMORY USAGE visitors
# Returns: approximately 12,304 bytes (12 KB)

# Same for 1 item or 1 billion items!
```

📖 [HyperLogLog Commands](https://redis.io/docs/latest/commands/?group=hyperloglog)

## Resources

- [HyperLogLog](https://redis.io/docs/latest/develop/data-types/probabilistic/hyperloglogs/) — Redis HyperLogLog documentation

---

> 📘 *This lesson is part of the [Probabilistic Data Structures and Analytics](https://stanza.dev/courses/redis-probabilistic) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*