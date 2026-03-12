---
source_course: "redis-probabilistic"
source_lesson: "redis-probabilistic-tdigest"
---

# t-digest: Percentile Estimation

## Introduction

Percentiles answer the questions that averages hide. "What's the 99th percentile response time?" tells you far more about user experience than the mean. t-digest computes accurate percentile estimates from streaming data using fixed memory, making it essential for latency monitoring and SLA tracking.

## Key Concepts

- **Quantile**: A value below which a given fraction of observations fall. The 0.95 quantile (95th percentile) is the value below which 95% of data points lie.
- **Compression parameter**: Controls the accuracy-memory trade-off. Higher compression means more accuracy and more memory. Default is 100; values of 200-500 are common in production.
- **CDF (Cumulative Distribution Function)**: The inverse of quantile — given a value, CDF tells you what fraction of observations are less than or equal to it.
- **Centroid**: t-digest internally groups nearby values into centroids (weighted clusters). Centroids near the tails (extremes) are kept smaller for higher accuracy where it matters most.

## Real World Context

Every production system needs latency percentiles. SLAs are defined in terms of p50, p95, and p99. Without t-digest, computing percentiles requires storing every single observation and sorting them — an O(n log n) memory and time cost that grows without bound.

## Deep Dive

### Creating a t-digest

```redis
# Default compression (100)
TDIGEST.CREATE latency

# Higher compression for more accuracy
TDIGEST.CREATE latency COMPRESSION 500
```

### Adding Values

```redis
# Add individual values
TDIGEST.ADD latency 45.2 52.1 38.7 61.3 49.8

# Values are added to the streaming digest immediately
```

### Querying Percentiles with TDIGEST.QUANTILE

```redis
# Get the median (50th percentile)
TDIGEST.QUANTILE latency 0.5
# Returns: [48.5]

# Get multiple percentiles at once
TDIGEST.QUANTILE latency 0.5 0.9 0.95 0.99
# Returns: [48.5, 95.2, 120.1, 128.7]
```

### Querying the CDF with TDIGEST.CDF

```redis
# What fraction of values are ≤ 50ms?
TDIGEST.CDF latency 50
# Returns: [0.52] → 52% of values are ≤ 50ms

# Multiple values
TDIGEST.CDF latency 50 100 150
# Returns: [0.52, 0.89, 0.97]
```

### Other Useful Commands

```redis
# Min and max values
TDIGEST.MIN latency
TDIGEST.MAX latency

# Trimmed mean (exclude outliers)
TDIGEST.TRIMMED_MEAN latency 0.1 0.9
# Mean of values between 10th and 90th percentiles

# Merge multiple digests
TDIGEST.MERGE combined COMPRESSION 200 2 latency1 latency2
```

### Practical Example: API Latency Monitor

```python
import redis

r = redis.Redis()

class LatencyMonitor:
    def __init__(self, name, compression=200):
        self.key = f'latency:{name}'
        try:
            r.execute_command('TDIGEST.CREATE', self.key, 'COMPRESSION', compression)
        except redis.ResponseError:
            pass

    def record(self, latency_ms):
        """Record a latency measurement."""
        r.execute_command('TDIGEST.ADD', self.key, latency_ms)

    def get_percentiles(self):
        """Get standard SLA percentiles."""
        result = r.execute_command(
            'TDIGEST.QUANTILE', self.key, 0.5, 0.9, 0.95, 0.99
        )
        return {'p50': result[0], 'p90': result[1], 'p95': result[2], 'p99': result[3]}

    def check_sla(self, threshold_ms):
        """What percentage of requests are under the SLA threshold?"""
        result = r.execute_command('TDIGEST.CDF', self.key, threshold_ms)
        return round(result[0] * 100, 2)

monitor = LatencyMonitor('api:users')
for ms in [45, 52, 48, 120, 55, 42, 67, 51, 95, 110]:
    monitor.record(ms)

print(monitor.get_percentiles())  # {'p50': 51.5, 'p90': 115.0, ...}
print(f"{monitor.check_sla(100)}% of requests under 100ms SLA")
```

### Time-Windowed Percentiles

Create per-minute digests and merge them for broader windows:

```python
from datetime import datetime, timedelta

def record_latency_windowed(latency_ms):
    """Record latency in the current minute's t-digest."""
    minute = datetime.utcnow().strftime('%Y-%m-%d:%H:%M')
    key = f'latency:{minute}'
    try:
        r.execute_command('TDIGEST.CREATE', key)
        r.expire(key, 3600)
    except redis.ResponseError:
        pass
    r.execute_command('TDIGEST.ADD', key, latency_ms)

def get_hourly_percentiles():
    """Merge last 60 minutes and get percentiles."""
    now = datetime.utcnow()
    keys = []
    for i in range(60):
        minute = (now - timedelta(minutes=i)).strftime('%Y-%m-%d:%H:%M')
        key = f'latency:{minute}'
        if r.exists(key):
            keys.append(key)
    if not keys:
        return None
    r.execute_command('TDIGEST.MERGE', 'latency:hourly_tmp',
                      'COMPRESSION', 200, len(keys), *keys)
    result = r.execute_command('TDIGEST.QUANTILE',
                               'latency:hourly_tmp', 0.5, 0.95, 0.99)
    r.delete('latency:hourly_tmp')
    return {'p50': result[0], 'p95': result[1], 'p99': result[2]}
```

## Common Pitfalls

1. **Confusing QUANTILE and CDF** — QUANTILE takes a fraction (0.95) and returns a value. CDF takes a value and returns a fraction. They are inverses of each other.
2. **Using default compression for tail-sensitive workloads** — The default compression of 100 is fine for p50 but may lose accuracy at p99 and beyond. Use COMPRESSION 500 for SLA-critical monitoring.

## Best Practices

1. **Use TDIGEST.TRIMMED_MEAN to ignore outliers** — When a single 10-second timeout skews your average, trimmed mean between the 5th and 95th percentiles gives a more representative central tendency.
2. **Merge per-window digests for flexible time ranges** — Rather than one ever-growing digest, create per-minute or per-hour digests and merge on demand. This lets you query any arbitrary time range.

## Summary

- t-digest estimates percentiles from streaming data in fixed memory using the COMPRESSION parameter.
- TDIGEST.QUANTILE returns values at given percentile ranks; TDIGEST.CDF does the inverse.
- Higher compression (200-500) improves accuracy at the tails (p99, p99.9) where SLAs are defined.
- Time-windowed digests with TDIGEST.MERGE enable flexible historical percentile queries.

## Code Examples

**API latency monitoring with TDIGEST.QUANTILE for percentiles and TDIGEST.CDF for SLA compliance**

```python
import redis

r = redis.Redis()

# Create a t-digest for API latency monitoring
try:
    r.execute_command('TDIGEST.CREATE', 'latency:api', 'COMPRESSION', 200)
except redis.ResponseError:
    pass

# Record latency samples
latencies = [45, 52, 48, 120, 55, 42, 67, 51, 95, 110, 88, 73]
for ms in latencies:
    r.execute_command('TDIGEST.ADD', 'latency:api', ms)

# Get SLA percentiles
result = r.execute_command('TDIGEST.QUANTILE', 'latency:api', 0.5, 0.95, 0.99)
print(f'p50={result[0]:.1f}ms, p95={result[1]:.1f}ms, p99={result[2]:.1f}ms')

# Check what percentage of requests are under 100ms
cdf = r.execute_command('TDIGEST.CDF', 'latency:api', 100)
print(f'{cdf[0] * 100:.1f}% of requests are under 100ms')
```


## Resources

- [t-digest](https://redis.io/docs/latest/develop/data-types/probabilistic/t-digest/) — Redis t-digest documentation
- [t-digest Commands](https://redis.io/docs/latest/commands/?group=tdigest) — Full reference for all t-digest commands

---

> 📘 *This lesson is part of the [Probabilistic Data Structures and Analytics](https://stanza.dev/courses/redis-probabilistic) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*