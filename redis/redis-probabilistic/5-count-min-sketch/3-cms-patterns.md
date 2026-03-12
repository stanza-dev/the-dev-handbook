---
source_course: "redis-probabilistic"
source_lesson: "redis-probabilistic-cms-patterns"
---

# CMS Production Patterns

## Introduction

Knowing CMS commands is one thing; deploying CMS in production is another. This lesson covers battle-tested patterns for network traffic monitoring, click counting, query popularity tracking, and merging sketches across time windows.

## Key Concepts

- **Time-windowed sketches**: Creating separate CMS instances per time period (minute, hour, day) and merging them for aggregate analysis.
- **Sketch merging**: Combining multiple CMS instances into one using CMS.MERGE, with optional weights for time-decay or importance.
- **Layered counting**: Using CMS alongside other structures (Bloom for dedup, Top-K for ranking) to build complete analytics pipelines.

## Real World Context

Every major content platform needs to answer questions like "how many times was this video watched today?" or "which API endpoints are getting hammered?" CMS lets you answer these questions in kilobytes instead of gigabytes, even for millions of distinct items.

## Deep Dive

### Pattern 1: Network Traffic Monitoring

Track packet counts per source IP without storing every IP:

```python
import redis
from datetime import datetime

r = redis.Redis()

def track_packet(source_ip):
    """Record a packet from source_ip in current minute's sketch."""
    minute = datetime.utcnow().strftime('%Y%m%d%H%M')
    key = f'traffic:{minute}'
    try:
        r.execute_command('CMS.INITBYPROB', key, 0.001, 0.01)
        r.expire(key, 7200)  # Keep 2 hours of per-minute sketches
    except redis.ResponseError:
        pass
    r.execute_command('CMS.INCRBY', key, source_ip, 1)

def get_ip_traffic(source_ip, minutes_back=60):
    """Estimate total packets from an IP over the last N minutes."""
    now = datetime.utcnow()
    total = 0
    for i in range(minutes_back):
        minute = (now - timedelta(minutes=i)).strftime('%Y%m%d%H%M')
        key = f'traffic:{minute}'
        try:
            result = r.execute_command('CMS.QUERY', key, source_ip)
            total += result[0]
        except redis.ResponseError:
            continue  # Sketch doesn't exist for that minute
    return total
```

### Pattern 2: Click Counting with Time Decay

Merge hourly sketches with decaying weights to emphasize recent clicks:

```python
def merge_with_decay(output_key, hourly_keys, decay=0.9):
    """Merge hourly sketches with exponential time decay.
    Most recent hour gets weight 1.0, previous gets 0.9, etc."""
    existing_keys = [k for k in hourly_keys if r.exists(k)]
    if not existing_keys:
        return

    weights = [decay ** i for i in range(len(existing_keys))]
    weights.reverse()  # Most recent last in list = highest weight

    args = [output_key, len(existing_keys)] + existing_keys + ['WEIGHTS'] + weights
    r.execute_command('CMS.MERGE', *args)
```

### Pattern 3: Query Popularity Tracking

Track search query frequency to power autocomplete ranking:

```python
class QueryPopularity:
    def __init__(self):
        self.daily_key = f'queries:{datetime.utcnow().strftime("%Y%m%d")}'
        try:
            r.execute_command('CMS.INITBYPROB', self.daily_key, 0.0001, 0.01)
            r.expire(self.daily_key, 86400 * 7)  # Keep 7 days
        except redis.ResponseError:
            pass

    def record_query(self, query_text):
        """Record a search query."""
        normalized = query_text.strip().lower()
        r.execute_command('CMS.INCRBY', self.daily_key, normalized, 1)

    def get_query_count(self, query_text):
        """Get estimated count for a query."""
        normalized = query_text.strip().lower()
        result = r.execute_command('CMS.QUERY', self.daily_key, normalized)
        return result[0]

    def merge_weekly(self):
        """Merge last 7 daily sketches into a weekly aggregate."""
        keys = []
        for i in range(7):
            day = (datetime.utcnow() - timedelta(days=i)).strftime('%Y%m%d')
            key = f'queries:{day}'
            if r.exists(key):
                keys.append(key)
        if keys:
            week = datetime.utcnow().strftime('%Y-W%W')
            r.execute_command('CMS.MERGE', f'queries:week:{week}', len(keys), *keys)
```

### Pattern 4: Merging Across Distributed Nodes

When running multiple application servers, each can maintain a local CMS. Periodically merge them:

```redis
# Each server writes to its own sketch
CMS.INCRBY clicks:server1 "/home" 1
CMS.INCRBY clicks:server2 "/home" 1

# Periodically merge all server sketches
CMS.MERGE clicks:combined 2 clicks:server1 clicks:server2

# Query the merged sketch
CMS.QUERY clicks:combined "/home"
```

## Common Pitfalls

1. **Merging sketches with different dimensions** — CMS.MERGE requires all source sketches to have the same width and depth. Always use the same initialization parameters across sketches you plan to merge.
2. **Not setting TTL on time-windowed sketches** — Without EXPIRE, old sketches accumulate forever. Always set a TTL when creating time-bucketed sketches.

## Best Practices

1. **Use consistent naming conventions** — Embed the time granularity in the key name (e.g., `cms:clicks:20250312:14` for hour 14 of March 12). This makes it easy to enumerate and merge.
2. **Create sketches idempotently** — Wrap CMS.INITBYPROB in a try/except since it errors if the key already exists. This makes your code safe for concurrent workers.
3. **Monitor with CMS.INFO** — Periodically check the count field in CMS.INFO to ensure your sketches are receiving data and to estimate total stream size.

## Summary

- Time-windowed sketches (per minute, hour, or day) keep absolute error bounded and enable flexible time-range queries via CMS.MERGE.
- CMS.MERGE with WEIGHTS enables time-decay patterns for recency-biased counting.
- Distributed systems can merge per-node sketches into a global view without shipping raw events.
- Always use consistent dimensions, set TTLs, and create sketches idempotently.

## Code Examples

**Hourly time-windowed CMS with daily merging for analytics event counting**

```python
import redis
from datetime import datetime, timedelta

r = redis.Redis()

def create_hourly_sketch():
    """Create a CMS for the current hour."""
    hour = datetime.utcnow().strftime('%Y%m%d%H')
    key = f'analytics:{hour}'
    try:
        r.execute_command('CMS.INITBYPROB', key, 0.001, 0.01)
        r.expire(key, 86400)  # Keep for 24 hours
    except redis.ResponseError:
        pass
    return key

def record_event(event_name):
    """Record an event in the current hour's sketch."""
    key = create_hourly_sketch()
    r.execute_command('CMS.INCRBY', key, event_name, 1)

def get_daily_count(event_name):
    """Merge last 24 hours and query an event's count."""
    keys = []
    now = datetime.utcnow()
    for i in range(24):
        hour = (now - timedelta(hours=i)).strftime('%Y%m%d%H')
        key = f'analytics:{hour}'
        if r.exists(key):
            keys.append(key)
    if not keys:
        return 0
    r.execute_command('CMS.MERGE', 'analytics:daily_tmp', len(keys), *keys)
    result = r.execute_command('CMS.QUERY', 'analytics:daily_tmp', event_name)
    r.delete('analytics:daily_tmp')
    return result[0]
```


## Resources

- [Count-Min Sketch](https://redis.io/docs/latest/develop/data-types/probabilistic/count-min-sketch/) — Redis Count-Min Sketch documentation
- [CMS.MERGE Command](https://redis.io/docs/latest/commands/cms.merge/) — CMS.MERGE command reference for combining sketches

---

> 📘 *This lesson is part of the [Probabilistic Data Structures and Analytics](https://stanza.dev/courses/redis-probabilistic) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*