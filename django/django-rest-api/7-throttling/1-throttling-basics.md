---
source_course: "django-rest-api"
source_lesson: "django-rest-api-throttling-basics"
---

# Implementing Rate Limiting

## Introduction

Without rate limiting, a single client can monopolize your API, starving other users and potentially crashing your server. Rate limiting ensures fair usage and protects against abuse, bots, and denial-of-service attacks.

Rate limiting protects your API from abuse and ensures fair usage across all clients.


## Key Concepts

**Rate Limiting**: Restricting the number of API requests a client can make within a time window.

**Throttling**: The process of slowing down or rejecting requests that exceed the rate limit.

**429 Too Many Requests**: The HTTP status code returned when a rate limit is exceeded.

**Client Identification**: Identifying clients by IP address, API key, or user account for per-client limits.

## Real World Context

Rate limiting is essential for:
- **Public APIs**: Preventing abuse and ensuring fair usage across all consumers
- **Authentication endpoints**: Limiting login attempts to prevent brute-force attacks
- **Expensive operations**: Protecting CPU-intensive endpoints like search or report generation
- **Tiered pricing**: Offering different rate limits for free, pro, and enterprise plans

## Simple Throttling Decorator

This decorator tracks request counts per client using Django's cache framework and returns 429 when the limit is exceeded:

```python
from functools import wraps
from django.core.cache import cache
from django.http import JsonResponse
import time


def rate_limit(requests_per_minute=60):
    """Rate limit decorator."""
    def decorator(view_func):
        @wraps(view_func)
        def wrapper(request, *args, **kwargs):
            # Create unique key for this client
            client_ip = get_client_ip(request)
            key = f'rate_limit:{client_ip}:{view_func.__name__}'
            
            # Get current request count
            current = cache.get(key, {'count': 0, 'start': time.time()})
            
            # Reset if minute has passed
            if time.time() - current['start'] > 60:
                current = {'count': 0, 'start': time.time()}
            
            # Check limit
            if current['count'] >= requests_per_minute:
                return JsonResponse({
                    'error': 'Rate limit exceeded',
                    'retry_after': 60 - (time.time() - current['start'])
                }, status=429)
            
            # Increment count
            current['count'] += 1
            cache.set(key, current, 60)
            
            # Process request
            response = view_func(request, *args, **kwargs)
            
            # Add rate limit headers
            response['X-RateLimit-Limit'] = str(requests_per_minute)
            response['X-RateLimit-Remaining'] = str(requests_per_minute - current['count'])
            
            return response
        return wrapper
    return decorator


def get_client_ip(request):
    x_forwarded_for = request.META.get('HTTP_X_FORWARDED_FOR')
    if x_forwarded_for:
        return x_forwarded_for.split(',')[0].strip()
    return request.META.get('REMOTE_ADDR')
```

The decorator adds `X-RateLimit-Limit` and `X-RateLimit-Remaining` headers to every response, letting clients monitor their quota before hitting the limit.

## Using the Decorator

Apply different limits based on endpoint cost -- reads can be generous while writes should be stricter:

```python
@rate_limit(requests_per_minute=100)
def api_list_articles(request):
    articles = Article.objects.filter(status='published')
    return JsonResponse({'articles': list(articles.values())})


@rate_limit(requests_per_minute=10)  # Stricter limit for writes
def api_create_article(request):
    # ...
```

Write endpoints use 10 requests/minute while reads allow 100, reflecting the higher server cost of mutations.

## Token Bucket Algorithm

The token bucket allows short bursts above the average rate by accumulating tokens over time. Each request consumes a token, and tokens refill at a steady rate:

```python
import time
from django.core.cache import cache


class TokenBucket:
    """
    Token bucket rate limiter.
    
    Allows bursts up to max_tokens, refills at rate tokens/second.
    """
    def __init__(self, key, max_tokens, refill_rate):
        self.key = key
        self.max_tokens = max_tokens
        self.refill_rate = refill_rate
    
    def consume(self, tokens=1):
        now = time.time()
        bucket = cache.get(self.key, {
            'tokens': self.max_tokens,
            'last_update': now
        })
        
        # Refill tokens based on time elapsed
        elapsed = now - bucket['last_update']
        bucket['tokens'] = min(
            self.max_tokens,
            bucket['tokens'] + elapsed * self.refill_rate
        )
        bucket['last_update'] = now
        
        # Try to consume tokens
        if bucket['tokens'] >= tokens:
            bucket['tokens'] -= tokens
            cache.set(self.key, bucket, 3600)  # 1 hour TTL
            return True
        
        cache.set(self.key, bucket, 3600)
        return False
    
    def get_remaining(self):
        bucket = cache.get(self.key)
        if bucket:
            return int(bucket['tokens'])
        return self.max_tokens
```

The `consume()` method first refills tokens based on elapsed time, then checks if enough tokens are available. This allows a burst of `max_tokens` requests followed by a sustained rate of `refill_rate` per second.

## User-Based Rate Limiting

Different users deserve different limits. This decorator applies generous limits for authenticated users and even higher limits for premium subscribers:

```python
from django.http import JsonResponse


def user_rate_limit(anonymous_limit=30, authenticated_limit=100):
    """Different limits for anonymous vs authenticated users."""
    def decorator(view_func):
        @wraps(view_func)
        def wrapper(request, *args, **kwargs):
            if request.user.is_authenticated:
                key = f'rate:{request.user.id}'
                limit = authenticated_limit
                
                # Premium users get higher limits
                if hasattr(request.user, 'subscription'):
                    if request.user.subscription.plan == 'premium':
                        limit = 1000
            else:
                key = f'rate:{get_client_ip(request)}'
                limit = anonymous_limit
            
            bucket = TokenBucket(key, max_tokens=limit, refill_rate=limit/60)
            
            if not bucket.consume():
                return JsonResponse({
                    'error': 'Rate limit exceeded',
                    'limit': limit,
                    'remaining': bucket.get_remaining()
                }, status=429)
            
            return view_func(request, *args, **kwargs)
        return wrapper
    return decorator
```

Anonymous users are identified by IP address, while authenticated users are tracked by user ID, ensuring accurate per-user limits regardless of network configuration.

## Middleware Approach

For global rate limiting across all API endpoints, a middleware applies limits before any view code runs:

```python
# middleware.py
from django.http import JsonResponse
from django.core.cache import cache
import time


class RateLimitMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
    
    def __call__(self, request):
        # Only rate limit API endpoints
        if not request.path.startswith('/api/'):
            return self.get_response(request)
        
        client_ip = self.get_client_ip(request)
        key = f'rate_limit:{client_ip}'
        
        # Simple sliding window
        now = time.time()
        window = cache.get(key, [])
        
        # Remove old requests
        window = [t for t in window if now - t < 60]
        
        if len(window) >= 100:
            return JsonResponse({
                'error': 'Rate limit exceeded',
                'retry_after': 60 - (now - window[0])
            }, status=429)
        
        window.append(now)
        cache.set(key, window, 60)
        
        response = self.get_response(request)
        response['X-RateLimit-Remaining'] = str(100 - len(window))
        return response
    
    def get_client_ip(self, request):
        xff = request.META.get('HTTP_X_FORWARDED_FOR')
        return xff.split(',')[0].strip() if xff else request.META.get('REMOTE_ADDR')
```

The sliding window approach stores timestamps of recent requests, removing entries older than 60 seconds. This prevents the boundary-burst problem that fixed-window counters have.

## Handling 429 Responses in Clients

Clients should handle 429 responses gracefully by waiting for the `retry_after` period before retrying:

```javascript
async function apiRequest(url, options = {}) {
    const response = await fetch(url, options);
    
    if (response.status === 429) {
        const data = await response.json();
        const retryAfter = data.retry_after || 60;
        
        console.log(`Rate limited. Retrying in ${retryAfter} seconds...`);
        
        await new Promise(resolve => setTimeout(resolve, retryAfter * 1000));
        return apiRequest(url, options);  // Retry
    }
    
    return response;
}
```

The recursive retry waits for `retryAfter` seconds before sending the request again. In production, add a maximum retry count to prevent infinite loops.

## Common Pitfalls

1. **Rate limiting only by IP**: Behind a corporate proxy, thousands of users share one IP. Rate limit by authenticated user when possible.

2. **Not communicating limits**: Clients need rate limit headers to adjust their behavior proactively.

3. **Using in-memory cache across multiple servers**: Each server tracks independently, allowing N times the limit (use Redis for shared state).

## Best Practices

1. **Use different limits for different endpoints**: Login should be strict (5/min), reads can be generous (100/min).

2. **Include rate limit headers**: X-RateLimit-Limit, X-RateLimit-Remaining, X-RateLimit-Reset on every response.

3. **Use Redis for distributed rate limiting**: Shared state ensures correct limits across multiple servers.

4. **Higher limits for authenticated users**: Encourage registration by giving authenticated users more generous limits.

## Summary

Rate limiting protects your API from abuse and ensures fair usage. Implement it using Django's cache framework with decorators or middleware, communicate limits via standard headers, and use different thresholds for different endpoints and user tiers.

## Code Examples

**Simple rate limiting decorator using Django's cache framework**

```python
from functools import wraps
from django.core.cache import cache
from django.http import JsonResponse
import time

def rate_limit(requests_per_minute=60):
    def decorator(view_func):
        @wraps(view_func)
        def wrapper(request, *args, **kwargs):
            ip = request.META.get('REMOTE_ADDR')
            key = f'rate_limit:{ip}:{view_func.__name__}'
            current = cache.get(key, {'count': 0, 'start': time.time()})
            if time.time() - current['start'] > 60:
                current = {'count': 0, 'start': time.time()}
            if current['count'] >= requests_per_minute:
                return JsonResponse({'error': 'Rate limit exceeded'}, status=429)
            current['count'] += 1
            cache.set(key, current, 60)
            return view_func(request, *args, **kwargs)
        return wrapper
    return decorator
```


## Resources

- [Rate Limiting Patterns](https://cloud.google.com/architecture/rate-limiting-strategies-techniques) — Rate limiting strategies and techniques

---

> 📘 *This lesson is part of the [Django REST API Development](https://stanza.dev/courses/django-rest-api) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*