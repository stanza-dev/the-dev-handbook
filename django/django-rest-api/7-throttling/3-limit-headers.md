---
source_course: "django-rest-api"
source_lesson: "django-rest-api-rate-limit-headers"
---

# Rate Limit Headers and Responses

## Introduction

Good rate limiting isn't just about blocking requests—it's about communicating clearly with clients so they can adjust their behavior.

## Key Concepts

**X-RateLimit-Limit**: Maximum requests allowed in the window.

**X-RateLimit-Remaining**: Requests remaining in current window.

**X-RateLimit-Reset**: Timestamp when the limit resets.

**Retry-After**: Seconds to wait before retrying (on 429).

## Deep Dive

### Standard Headers

```python
def add_rate_limit_headers(response, limit, remaining, reset_time):
    response['X-RateLimit-Limit'] = str(limit)
    response['X-RateLimit-Remaining'] = str(max(0, remaining))
    response['X-RateLimit-Reset'] = str(int(reset_time))
    return response

def rate_limited_response(retry_after):
    response = JsonResponse({
        'error': {
            'code': 'RATE_LIMIT_EXCEEDED',
            'message': 'Too many requests. Please slow down.',
            'retry_after': retry_after
        }
    }, status=429)
    response['Retry-After'] = str(int(retry_after))
    return response
```

### Complete Rate Limiter

```python
class RateLimiter:
    def __init__(self, limit=100, window=60):
        self.limit = limit
        self.window = window
    
    def check(self, key):
        now = time.time()
        window_start = int(now // self.window) * self.window
        cache_key = f'rate:{key}:{window_start}'
        
        count = cache.get(cache_key, 0)
        remaining = self.limit - count - 1
        reset_time = window_start + self.window
        
        if count >= self.limit:
            return {
                'allowed': False,
                'limit': self.limit,
                'remaining': 0,
                'reset': reset_time,
                'retry_after': reset_time - now
            }
        
        cache.set(cache_key, count + 1, self.window + 1)
        return {
            'allowed': True,
            'limit': self.limit,
            'remaining': remaining,
            'reset': reset_time
        }
```

### Client-Side Handling

```javascript
async function makeRequest(url) {
    const response = await fetch(url);
    
    // Log rate limit status
    console.log('Rate limit:', {
        limit: response.headers.get('X-RateLimit-Limit'),
        remaining: response.headers.get('X-RateLimit-Remaining'),
        reset: new Date(response.headers.get('X-RateLimit-Reset') * 1000)
    });
    
    if (response.status === 429) {
        const retryAfter = response.headers.get('Retry-After') || 60;
        await sleep(retryAfter * 1000);
        return makeRequest(url);  // Retry
    }
    
    return response;
}
```

## Best Practices

1. **Always include headers**: Even on successful requests.
2. **Use standard header names**: X-RateLimit-* is widely recognized.
3. **Include Retry-After on 429**: Clients need to know when to retry.
4. **Log rate limit events**: Monitor for abuse patterns.

## Summary

Rate limit headers help clients self-regulate. Include limit, remaining, and reset headers on all responses, and provide retry-after on 429 errors. Clear communication prevents frustration and reduces support requests.

## Resources

- [HTTP 429 Status Code](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/429) — MDN documentation on 429 Too Many Requests

---

> 📘 *This lesson is part of the [Django REST API Development](https://stanza.dev/courses/django-rest-api) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*