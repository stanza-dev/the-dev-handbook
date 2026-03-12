---
source_course: "redis-probabilistic"
source_lesson: "redis-probabilistic-hll-merging"
---

# HyperLogLog Merging Patterns

## Introduction

One of HyperLogLog's most powerful properties is that two HLLs can be merged to produce a new HLL representing the union of both sets — without needing the original data. This makes PFMERGE the key to building scalable, time-windowed analytics with roll-up aggregation.

## Key Concepts

- **PFMERGE**: Combines multiple HyperLogLogs into a destination key. The result represents the union of all source HLLs.
- **Roll-up aggregation**: Merging fine-grained HLLs (e.g., hourly) into coarser ones (e.g., daily, weekly, monthly).
- **Union cardinality**: The count of distinct items across multiple sets, with duplicates counted only once.
- **Additive property**: HyperLogLog registers can be merged by taking the maximum value per register. This is why merging works without the original data.

## Real World Context

A SaaS analytics dashboard needs to show unique active users per hour, per day, per week, and per month. Storing user IDs for every time window at every granularity would consume enormous memory. With HyperLogLog merging, you store one 12 KB HLL per hour, merge 24 hourly HLLs into a daily HLL, 7 daily HLLs into a weekly HLL, and so on. Each level of aggregation costs only 12 KB.

## Deep Dive

### Basic Merging

PFMERGE takes a destination key and one or more source keys:

```redis
# Merge two daily HLLs into a weekend aggregate
PFMERGE visitors:weekend visitors:saturday visitors:sunday

# The destination now contains the union
PFCOUNT visitors:weekend
# Returns: unique visitors across both days
```

Important: PFMERGE overwrites the destination key. If the destination already exists, its previous contents are replaced.

### Time-Windowed Aggregation Pattern

The most common pattern is hierarchical roll-up:

```
Hour HLLs:   [H0] [H1] [H2] ... [H23]
                \   |   /
                 \  |  /
Daily HLL:      [Day 1]

7 Daily HLLs:  [D1] [D2] ... [D7]
                \   |   ...  /
Weekly HLL:    [Week 1]

4 Weekly HLLs: [W1] [W2] [W3] [W4]
                \   |    |   /
Monthly HLL:   [Month 1]
```

Implementation:

```python
import redis
from datetime import datetime, timedelta

r = redis.Redis()

def track_visitor(user_id):
    """Track visitor in hourly bucket."""
    now = datetime.utcnow()
    hour_key = now.strftime('visitors:%Y-%m-%d:%H')
    r.pfadd(hour_key, user_id)
    # Expire hourly keys after 48 hours
    r.expire(hour_key, 48 * 3600)

def rollup_daily(date_str):
    """Merge 24 hourly HLLs into a daily HLL."""
    hour_keys = [f'visitors:{date_str}:{h:02d}' for h in range(24)]
    existing = [k for k in hour_keys if r.exists(k)]
    if existing:
        daily_key = f'visitors:daily:{date_str}'
        r.pfmerge(daily_key, *existing)
        r.expire(daily_key, 35 * 86400)  # Keep 35 days
        return r.pfcount(daily_key)
    return 0

def rollup_weekly(year, week_num):
    """Merge 7 daily HLLs into a weekly HLL."""
    # Get the Monday of the given week
    monday = datetime.strptime(f'{year}-W{week_num:02d}-1', '%G-W%V-%u')
    daily_keys = []
    for i in range(7):
        day = (monday + timedelta(days=i)).strftime('%Y-%m-%d')
        daily_keys.append(f'visitors:daily:{day}')
    existing = [k for k in daily_keys if r.exists(k)]
    if existing:
        weekly_key = f'visitors:weekly:{year}-W{week_num:02d}'
        r.pfmerge(weekly_key, *existing)
        r.expire(weekly_key, 370 * 86400)  # Keep ~1 year
        return r.pfcount(weekly_key)
    return 0
```

### PFMERGE vs Multi-Key PFCOUNT

Both can compute union cardinality, but they serve different purposes:

| | PFMERGE | Multi-key PFCOUNT |
|---|---------|-------------------|
| Stores result | Yes (destination key) | No (read-only) |
| Reusable | Yes | No (recomputed each time) |
| Use case | Roll-up aggregation | Ad-hoc queries |
| Side effect | Creates/overwrites key | None |

Use PFMERGE when you want to persist the aggregate for repeated reads. Use multi-key PFCOUNT for one-off queries.

### Merging Across Dimensions

You can also merge across non-time dimensions:

```python
def get_unique_visitors_across_pages(page_slugs):
    """Count unique visitors across multiple pages."""
    keys = [f'visitors:page:{slug}' for slug in page_slugs]
    existing = [k for k in keys if r.exists(k)]
    if not existing:
        return 0
    return r.pfcount(*existing)

def merge_regional_data(region_codes, date_str):
    """Merge regional visitor HLLs into a global view."""
    region_keys = [f'visitors:{region}:{date_str}' for region in region_codes]
    existing = [k for k in region_keys if r.exists(k)]
    if existing:
        global_key = f'visitors:global:{date_str}'
        r.pfmerge(global_key, *existing)
        return r.pfcount(global_key)
    return 0
```

### Error Accumulation in Merges

Merging HLLs does not significantly increase the error rate. The merged HLL has the same 0.81% standard error as any individual HLL because the merge operation takes the maximum register value — the same operation as if all items had been added to a single HLL from the start.

## Common Pitfalls

1. **Merging into a source key by accident** — If you do `PFMERGE visitors:mon visitors:mon visitors:tue`, the Monday data is overwritten with the merged result. Always use a separate destination key.
2. **Forgetting to check key existence** — Merging with non-existent keys is safe (they are treated as empty HLLs), but calling PFMERGE with a list of entirely non-existent keys creates an empty HLL at the destination.
3. **Not setting TTLs on intermediate keys** — Without expiration, hourly and daily HLLs accumulate and consume memory indefinitely. Always set TTLs after creating roll-up aggregates.

## Best Practices

1. **Run roll-ups on a schedule** — Use a cron job or scheduled task to merge hourly into daily, daily into weekly. Do not compute roll-ups on every dashboard request.
2. **Keep source keys until roll-up is confirmed** — Set hourly key TTLs longer than your roll-up interval (e.g., 48 hours for hourly keys that are rolled up daily) so you can re-run a failed roll-up.
3. **Use consistent UTC timestamps in keys** — Avoid timezone ambiguity by always using UTC in key names. This prevents double-counting or missed data at timezone boundaries.

## Summary

- PFMERGE combines multiple HyperLogLogs by taking the maximum register value per bucket, producing a union.
- Use hierarchical roll-ups (hourly to daily to weekly to monthly) for efficient time-windowed analytics.
- Merging does not increase the error rate — the merged HLL has the same 0.81% standard error.
- Always merge into a separate destination key to avoid overwriting source data.
- Set TTLs on granular keys and keep them slightly longer than your roll-up interval for safety.

## Code Examples

**Hierarchical roll-up: merging hourly HLLs into a daily aggregate**

```python
import redis
from datetime import datetime, timedelta

r = redis.Redis()

# Simulate hourly visitor tracking and daily roll-up

# Track visitors across 3 hours
for hour in range(3):
    key = f'visitors:2024-01-15:{hour:02d}'
    for user in range(100):
        # Some users visit multiple hours (overlap)
        r.pfadd(key, f'user:{user + hour * 50}')
    count = r.pfcount(key)
    print(f'Hour {hour}: ~{count} unique visitors')

# Roll up into daily aggregate
hour_keys = [f'visitors:2024-01-15:{h:02d}' for h in range(3)]
r.pfmerge('visitors:daily:2024-01-15', *hour_keys)

daily_count = r.pfcount('visitors:daily:2024-01-15')
print(f'Daily unique visitors: ~{daily_count}')
# Less than the sum of hourly counts due to overlap
```


## Resources

- [PFMERGE Command](https://redis.io/docs/latest/commands/pfmerge/) — PFMERGE command reference and behavior details
- [HyperLogLog](https://redis.io/docs/latest/develop/data-types/probabilistic/hyperloglogs/) — Redis HyperLogLog documentation

---

> 📘 *This lesson is part of the [Probabilistic Data Structures and Analytics](https://stanza.dev/courses/redis-probabilistic) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*