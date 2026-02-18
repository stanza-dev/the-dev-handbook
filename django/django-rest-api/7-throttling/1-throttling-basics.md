---
source_course: "django-rest-api"
source_lesson: "django-rest-api-throttling-basics"
---

# Implementing Rate Limiting

Rate limiting protects your API from abuse and ensures fair usage across all clients.

## Simple Throttling Decorator

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

## Using the Decorator

```python
@rate_limit(requests_per_minute=100)
def api_list_articles(request):
    articles = Article.objects.filter(status='published')
    return JsonResponse({'articles': list(articles.values())})


@rate_limit(requests_per_minute=10)  # Stricter limit for writes
def api_create_article(request):
    # ...
```

## Token Bucket Algorithm

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

## User-Based Rate Limiting

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

## Middleware Approach

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

## Handling 429 Responses in Clients

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

## Resources

- [Rate Limiting Patterns](https://cloud.google.com/architecture/rate-limiting-strategies-techniques) — Rate limiting strategies and techniques

---

> 📘 *This lesson is part of the [Django REST API Development](https://stanza.dev/courses/django-rest-api) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*