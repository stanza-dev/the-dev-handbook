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

## Deep Dive

### Fixed Window (Simplest)

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

### Tiered Limits by User

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

## Best Practices

1. **Different limits for different operations**: Login should have stricter limits than read operations.
2. **Higher limits for authenticated users**: Reward users who register.
3. **Communicate limits clearly**: Use X-RateLimit headers.
4. **Provide retry-after**: Tell clients when they can retry.

## Summary

Choose rate limiting strategies based on your needs: fixed window for simplicity, sliding window for accuracy, token bucket for burst tolerance. Apply different limits based on endpoint cost and user tier.

## Resources

- [Django Cache Framework](https://docs.djangoproject.com/en/6.0/topics/cache/) — Django caching for rate limiting

---

> 📘 *This lesson is part of the [Django REST API Development](https://stanza.dev/courses/django-rest-api) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*