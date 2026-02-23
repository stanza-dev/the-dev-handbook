---
source_course: "django-rest-api"
source_lesson: "django-rest-api-ddos-protection"
---

# DDoS Protection Basics

## Introduction

Rate limiting alone won't stop a determined attacker. Defense in depth with multiple layers protects your API from denial of service attacks.

## Key Concepts

**DDoS**: Distributed Denial of Service—overwhelming your server with requests from many sources.

**IP Blocking**: Blocking requests from specific IP addresses.

**WAF**: Web Application Firewall—filters malicious traffic.

**CDN**: Content Delivery Network—absorbs traffic at the edge.

## Real World Context

DDoS protection is critical for:
- **E-commerce during sales events**: Competitors or bots may try to crash your site
- **SaaS applications**: Service availability directly affects revenue and trust
- **Public APIs**: Open endpoints are easy targets for automated attacks
- **Financial services**: Downtime can result in regulatory penalties

## Deep Dive

### IP-Based Blocking

This middleware checks every incoming request against a set of blocked IPs stored in the cache. Blocked requests receive an immediate 403 response:

```python
# middleware.py
class IPBlockMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
        self.blocked_ips = cache.get('blocked_ips', set())
    
    def __call__(self, request):
        ip = get_client_ip(request)
        
        if ip in self.blocked_ips:
            return JsonResponse({'error': 'Blocked'}, status=403)
        
        return self.get_response(request)

def block_ip(ip, duration=3600):
    blocked = cache.get('blocked_ips', set())
    blocked.add(ip)
    cache.set('blocked_ips', blocked, duration)
```

The `block_ip()` function uses cache TTL to automatically unblock IPs after the specified duration, preventing permanent lockouts from temporary issues.

### Automatic Blocking

This system automatically escalates from rate limiting to full IP blocking when a client repeatedly exceeds their quota:

```python
def check_and_block(request):
    ip = get_client_ip(request)
    key = f'violations:{ip}'
    
    violations = cache.get(key, 0)
    
    if violations >= 10:  # 10 rate limit violations
        block_ip(ip, duration=3600)  # Block for 1 hour
        return True
    
    return False

def rate_limit_middleware(get_response):
    def middleware(request):
        if not rate_limiter.check(request):
            violations_key = f'violations:{get_client_ip(request)}'
            cache.set(violations_key, cache.get(violations_key, 0) + 1, 300)
            
            if check_and_block(request):
                return JsonResponse({'error': 'Blocked for abuse'}, status=403)
            
            return JsonResponse({'error': 'Rate limited'}, status=429)
        
        return get_response(request)
    return middleware
```

After 10 rate limit violations within 5 minutes (`cache TTL = 300`), the IP is blocked for one hour. This deters automated abuse while giving legitimate users a reasonable error recovery window.

### Defense Layers

1. **CDN/Edge**: Cloudflare, AWS CloudFront absorb traffic
2. **Load Balancer**: Distribute traffic, basic rate limiting
3. **WAF**: Block known attack patterns
4. **Application**: Rate limiting, authentication
5. **Database**: Connection pooling, query limits

## Common Pitfalls

1. **Relying only on application-level protection**: Application rate limiting can't handle volumetric attacks. Use CDN/WAF services.

2. **Blocking by IP alone**: Attackers use botnets with thousands of IPs. IP blocking is insufficient against distributed attacks.

3. **No incident response plan**: Knowing what to do during an attack is as important as prevention.

## Best Practices

1. **Use a CDN**: Absorbs attack traffic at the edge.
2. **Implement at multiple layers**: Don't rely on one defense.
3. **Monitor and alert**: Detect attacks early.
4. **Have a runbook**: Know what to do during an attack.

## Summary

DDoS protection requires multiple layers. Use CDN/WAF services for edge protection, implement application-level rate limiting and blocking, and have monitoring and response procedures in place.

## Code Examples

**IP blocking middleware for DDoS protection**

```python
from django.core.cache import cache
from django.http import JsonResponse

class IPBlockMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response

    def __call__(self, request):
        ip = request.META.get('REMOTE_ADDR')
        if cache.get(f'blocked:{ip}'):
            return JsonResponse({'error': 'Blocked'}, status=403)
        return self.get_response(request)

def block_ip(ip, duration=3600):
    cache.set(f'blocked:{ip}', True, duration)
```


## Resources

- [Django Security Middleware](https://docs.djangoproject.com/en/6.0/ref/middleware/#module-django.middleware.security) — Django security middleware

---

> 📘 *This lesson is part of the [Django REST API Development](https://stanza.dev/courses/django-rest-api) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*