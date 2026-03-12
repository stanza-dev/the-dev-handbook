---
source_course: "redis-probabilistic"
source_lesson: "redis-probabilistic-combining-structures"
---

# Combining Probabilistic Structures

## Introduction

Individual probabilistic structures are powerful, but the real magic happens when you combine them into pipelines. A Bloom filter for deduplication feeding into a CMS for counting, topped with a Top-K for trending and a t-digest for latency — this is how production analytics systems work at scale.

## Key Concepts

- **Analytics pipeline**: A chain of probabilistic structures where each stage handles a different aspect of data processing — deduplication, counting, ranking, and distribution analysis.
- **Deduplication layer**: Using a Bloom filter to ensure each event is only counted once, preventing inflated metrics from retries or duplicate deliveries.
- **Multi-structure synergy**: Each structure compensates for the others' limitations. Bloom filters cannot count; CMS cannot rank; Top-K cannot give percentiles. Together, they cover the full analytics spectrum.

## Real World Context

Consider a real-time analytics platform processing clickstream data. Every click needs to be deduplicated (Bloom), counted per URL (CMS), ranked for trending (Top-K), and measured for response time (t-digest). Doing all of this with exact data structures would require terabytes. With probabilistic structures, it fits in megabytes.

## Deep Dive

### Architecture: The Analytics Pipeline

```
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│  Bloom   │───▶│   CMS    │───▶│  Top-K   │    │ t-digest │
│  Filter  │    │ (counts) │    │(trending)│    │(latency) │
│ (dedup)  │    │          │    │          │    │          │
└──────────┘    └──────────┘    └──────────┘    └──────────┘
     │                │               │               │
  "Seen this    "How many       "What's          "What's the
   event?"      times?"        trending?"        p99?"
```

### Full Pipeline Implementation

```python
import redis
import time
import uuid

r = redis.Redis(decode_responses=True)

class AnalyticsPipeline:
    def __init__(self, name):
        self.name = name
        self._init_structures()

    def _init_structures(self):
        """Initialize all probabilistic structures."""
        # Bloom filter for deduplication
        try:
            r.execute_command('BF.RESERVE', f'{self.name}:dedup',
                            0.001, 10000000)  # 0.1% FP, 10M capacity
        except redis.ResponseError:
            pass

        # Count-Min Sketch for frequency counting
        try:
            r.execute_command('CMS.INITBYPROB', f'{self.name}:counts',
                            0.001, 0.01)
        except redis.ResponseError:
            pass

        # Top-K for trending items
        try:
            r.execute_command('TOPK.RESERVE', f'{self.name}:trending', 20)
        except redis.ResponseError:
            pass

        # t-digest for latency percentiles
        try:
            r.execute_command('TDIGEST.CREATE', f'{self.name}:latency',
                            'COMPRESSION', 200)
        except redis.ResponseError:
            pass

    def process_event(self, event_id, item_name, latency_ms=None):
        """Process a single event through the full pipeline."""
        # Stage 1: Deduplicate with Bloom filter
        is_new = r.execute_command('BF.ADD', f'{self.name}:dedup', event_id)
        if not is_new:
            return {'status': 'duplicate', 'event_id': event_id}

        # Stage 2: Count with CMS
        r.execute_command('CMS.INCRBY', f'{self.name}:counts', item_name, 1)

        # Stage 3: Update trending with Top-K
        displaced = r.execute_command('TOPK.ADD',
                                      f'{self.name}:trending', item_name)

        # Stage 4: Record latency if provided
        if latency_ms is not None:
            r.execute_command('TDIGEST.ADD',
                            f'{self.name}:latency', latency_ms)

        return {'status': 'processed', 'event_id': event_id,
                'displaced': displaced}

    def get_item_count(self, item_name):
        """Get estimated frequency for an item."""
        result = r.execute_command('CMS.QUERY',
                                   f'{self.name}:counts', item_name)
        return result[0]

    def get_trending(self):
        """Get current trending items with counts."""
        result = r.execute_command('TOPK.LIST',
                                   f'{self.name}:trending', 'WITHCOUNT')
        return {result[i]: int(result[i+1])
                for i in range(0, len(result), 2)}

    def get_latency_percentiles(self):
        """Get latency percentiles."""
        result = r.execute_command('TDIGEST.QUANTILE',
                                   f'{self.name}:latency',
                                   0.5, 0.9, 0.95, 0.99)
        return {'p50': result[0], 'p90': result[1],
                'p95': result[2], 'p99': result[3]}

    def get_dashboard(self):
        """Get a complete analytics dashboard."""
        return {
            'trending': self.get_trending(),
            'latency': self.get_latency_percentiles()
        }
```

### Using the Pipeline

```python
pipeline = AnalyticsPipeline('clickstream')

# Simulate events
events = [
    ('evt-001', '/home', 45),
    ('evt-002', '/products', 52),
    ('evt-003', '/home', 38),
    ('evt-001', '/home', 45),   # Duplicate — will be skipped
    ('evt-004', '/checkout', 120),
    ('evt-005', '/home', 55),
    ('evt-006', '/products', 42),
]

for event_id, page, latency in events:
    result = pipeline.process_event(event_id, page, latency)
    print(f"{event_id}: {result['status']}")

# Get dashboard
dashboard = pipeline.get_dashboard()
print(f"Trending: {dashboard['trending']}")
print(f"p99 latency: {dashboard['latency']['p99']}ms")
```

### Memory Budget Comparison

```
┌─────────────────────────────────────────────────────────┐
│  Structure        │  Purpose      │  Memory (10M items) │
│  ─────────────────┼───────────────┼───────────────────  │
│  Bloom Filter     │  Dedup        │  ~12 MB             │
│  Count-Min Sketch │  Counting     │  ~54 KB             │
│  Top-K (20)       │  Trending     │  ~5 KB              │
│  t-digest         │  Percentiles  │  ~10 KB             │
│  ─────────────────┼───────────────┼───────────────────  │
│  Total            │               │  ~12.1 MB           │
│  Exact equivalent │               │  ~360 MB+           │
└─────────────────────────────────────────────────────────┘
```

## Common Pitfalls

1. **Skipping the deduplication layer** — Without Bloom filter dedup, retries and duplicate events inflate CMS counts and skew Top-K rankings. Always deduplicate before counting.
2. **Using one structure where you need two** — CMS cannot tell you which items are most frequent (it requires you to query specific items). You need Top-K for ranking. Similarly, Top-K does not give you percentile distributions — you need t-digest.

## Best Practices

1. **Initialize all structures idempotently** — Wrap every RESERVE/CREATE in a try/except so your pipeline startup is safe for restarts and concurrent workers.
2. **Separate concerns by key prefix** — Use consistent naming like `{pipeline}:dedup`, `{pipeline}:counts`, `{pipeline}:trending` to keep structures organized and easy to manage.
3. **Add latency tracking at the edge** — Record response time in t-digest at the point where you measure it (e.g., middleware), not inside the analytics pipeline itself.

## Summary

- Combining Bloom (dedup) + CMS (counting) + Top-K (trending) + t-digest (percentiles) creates a complete analytics pipeline in megabytes of memory.
- Each structure handles a different analytical question: existence, frequency, ranking, and distribution.
- Deduplication with Bloom filters before counting is critical for accurate downstream metrics.
- The total memory for 10 million events is roughly 12 MB — compared to 360+ MB for exact equivalents.

## Code Examples

**Complete analytics pipeline combining Bloom, CMS, Top-K, and t-digest for clickstream analysis**

```python
import redis

r = redis.Redis(decode_responses=True)

# Initialize the four structures
try:
    r.execute_command('BF.RESERVE', 'pipeline:dedup', 0.001, 1000000)
except redis.ResponseError:
    pass
try:
    r.execute_command('CMS.INITBYPROB', 'pipeline:counts', 0.001, 0.01)
except redis.ResponseError:
    pass
try:
    r.execute_command('TOPK.RESERVE', 'pipeline:trending', 10)
except redis.ResponseError:
    pass
try:
    r.execute_command('TDIGEST.CREATE', 'pipeline:latency', 'COMPRESSION', 200)
except redis.ResponseError:
    pass

def process_click(event_id, page_url, latency_ms):
    """Full pipeline: dedup → count → trend → latency."""
    # Dedup
    if not r.execute_command('BF.ADD', 'pipeline:dedup', event_id):
        return 'duplicate'
    # Count
    r.execute_command('CMS.INCRBY', 'pipeline:counts', page_url, 1)
    # Trend
    r.execute_command('TOPK.ADD', 'pipeline:trending', page_url)
    # Latency
    r.execute_command('TDIGEST.ADD', 'pipeline:latency', latency_ms)
    return 'processed'

# Process events
for eid, url, ms in [('e1','/home',45), ('e2','/shop',80), ('e1','/home',45)]:
    print(f'{eid}: {process_click(eid, url, ms)}')

# Query results
print('Trending:', r.execute_command('TOPK.LIST', 'pipeline:trending', 'WITHCOUNT'))
print('p95:', r.execute_command('TDIGEST.QUANTILE', 'pipeline:latency', 0.95))
```


## Resources

- [Probabilistic Data Types](https://redis.io/docs/latest/develop/data-types/probabilistic/) — Overview of all Redis probabilistic data structures

---

> 📘 *This lesson is part of the [Probabilistic Data Structures and Analytics](https://stanza.dev/courses/redis-probabilistic) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*