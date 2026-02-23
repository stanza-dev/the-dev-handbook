---
source_course: "django-rest-api"
source_lesson: "django-rest-api-rate-limit-strategies"
---

# Rate Limiting Strategies

## Introduction

Different rate limiting strategies serve different purposes. Understanding when to use each helps you build APIs that are both protected and user-friendly.

## Key Concepts

**Fixed Window**: Count requests in fixed time periods (e.g., per minute).

**Sliding Window**: Rolling window that provides smoother limiting.

**Token Bucket**: Allows bursts while maintaining average rate.

**Leaky Bucket**: Smooths out request rate by processing at a constant rate.

## Real World Context

Different strategies suit different scenarios:
- **Fixed window**: Simple APIs with predictable traffic patterns
- **Sliding window**: APIs needing smooth, consistent rate enforcement
- **Token bucket**: APIs that need to handle legitimate bursts (e.g., batch operations)
- **Leaky bucket**: APIs requiring constant throughput (e.g., message queues)

## Deep Dive

### Fixed Window (Simplest)

The fixed window approach counts requests within discrete time blocks (e.g., each minute). The cache key includes the window timestamp so the counter resets automatically:

```python
def fixed_window_limit(request, limit=100, window=60):
    key = f'rate:{get_client_id(request)}:{int(time.time() // window)}'
    count = cache.get(key, 0)
    
    if count >= limit:
        return False
    
    cache.set(key, count + 1, window)
    return True
```

**Pros**: Simple, predictable
**Cons**: Burst at window boundaries (200 requests if timed right)

### Sliding Window Log

The sliding window approach stores the timestamp of every recent request, providing smooth rate enforcement without boundary-burst issues:

```python
def sliding_window_limit(request, limit=100, window=60):
    key = f'rate:{get_client_id(request)}'
    now = time.time()
    
    # Get timestamps of recent requests
    timestamps = cache.get(key, [])
    timestamps = [t for t in timestamps if now - t < window]
    
    if len(timestamps) >= limit:
        return False
    
    timestamps.append(now)
    cache.set(key, timestamps, window)
    return True
```

**Pros**: No boundary burst issue
**Cons**: Higher memory usage (stores all timestamps)

### Per-Endpoint Limits

Different endpoints have different costs. Define a configuration dict that maps view names to their limits:

```python
RATE_LIMITS = {
    'default': (100, 60),      # 100/min
    'api_login': (5, 60),       # 5/min - prevent brute force
    'api_search': (30, 60),     # 30/min - expensive operation
    'api_export': (10, 3600),   # 10/hour - very expensive
}

def get_rate_limit(view_name):
    return RATE_LIMITS.get(view_name, RATE_LIMITS['default'])
```

Expensive operations like search and export get stricter limits, while the default covers standard CRUD endpoints. The tuple format is `(max_requests, window_seconds)`.

### Tiered Limits by User

Return different limits based on the user's subscription tier, enabling a freemium pricing model for your API:

```python
def get_user_limit(user):
    if not user.is_authenticated:
        return 30  # Anonymous
    
    tier = getattr(user, 'api_tier', 'free')
    limits = {
        'free': 100,
        'pro': 1000,
        'enterprise': 10000,
    }
    return limits.get(tier, 100)
```

Anonymous users get the lowest limit (30), while enterprise customers get 10,000. This function integrates with any of the rate limiting algorithms above.

## Common Pitfalls

1. **Fixed window boundary bursts**: A client can send 100 requests at 11:59:59 and 100 more at 12:00:00, getting 200 in 2 seconds.

2. **Sliding window memory usage**: Storing timestamps for every request uses more memory than simple counters.

3. **Wrong algorithm choice**: Using token bucket for login attempts (where bursts are bad) or fixed window for batch APIs (where bursts are normal).

## Best Practices

1. **Different limits for different operations**: Login should have stricter limits than read operations.
2. **Higher limits for authenticated users**: Reward users who register.
3. **Communicate limits clearly**: Use X-RateLimit headers.
4. **Provide retry-after**: Tell clients when they can retry.

## Summary

Choose rate limiting strategies based on your needs: fixed window for simplicity, sliding window for accuracy, token bucket for burst tolerance. Apply different limits based on endpoint cost and user tier.

## Code Examples

**Token bucket algorithm for rate limiting with burst support**

```python
import time
from django.core.cache import cache

class TokenBucket:
    def __init__(self, key, max_tokens, refill_rate):
        self.key = key
        self.max_tokens = max_tokens
        self.refill_rate = refill_rate

    def consume(self, tokens=1):
        now = time.time()
        bucket = cache.get(self.key, {'tokens': self.max_tokens, 'last_update': now})
        elapsed = now - bucket['last_update']
        bucket['tokens'] = min(self.max_tokens, bucket['tokens'] + elapsed * self.refill_rate)
        bucket['last_update'] = now
        if bucket['tokens'] >= tokens:
            bucket['tokens'] -= tokens
            cache.set(self.key, bucket, 3600)
            return True
        cache.set(self.key, bucket, 3600)
        return False
```


## Resources

- [Django Cache Framework](https://docs.djangoproject.com/en/6.0/topics/cache/) — Django caching for rate limiting

---

> 📘 *This lesson is part of the [Django REST API Development](https://stanza.dev/courses/django-rest-api) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*