---
source_course: "django-security"
source_lesson: "django-security-brute-force"
---

# Brute Force Protection

## Introduction

Brute force attacks try many passwords until one works. Without rate limiting, attackers can try thousands of combinations per minute.

## Key Concepts

**Rate Limiting**: Restricting how many attempts can be made in a time period.

**Account Lockout**: Temporarily disabling accounts after failed attempts.

**Progressive Delays**: Increasing wait time after each failure.



## Real World Context

Credential stuffing attacks use billions of leaked username/password pairs from previous breaches to try logging into other services. Without rate limiting, an attacker can test thousands of credentials per minute against your login endpoint using automated tools.

## Deep Dive

### Using django-axes

```python
# pip install django-axes

# settings.py
INSTALLED_APPS = ['axes', ...]
MIDDLEWARE = ['axes.middleware.AxesMiddleware', ...]

AUTHENTICATION_BACKENDS = [
    'axes.backends.AxesStandaloneBackend',
    'django.contrib.auth.backends.ModelBackend',
]

# Configuration
AXES_FAILURE_LIMIT = 5          # Lock after 5 failures
AXES_COOLOFF_TIME = 0.5         # Lock for 30 minutes
AXES_LOCKOUT_PARAMETERS = ['username', 'ip_address']
AXES_RESET_ON_SUCCESS = True
```

### Custom Rate Limiting

```python
from django.core.cache import cache
from django.http import HttpResponseForbidden

def rate_limit_login(request):
    ip = get_client_ip(request)
    key = f'login_attempts:{ip}'
    attempts = cache.get(key, 0)
    
    if attempts >= 5:
        return HttpResponseForbidden('Too many attempts. Try later.')
    
    # On failure
    cache.set(key, attempts + 1, 900)  # 15 minutes
    
    # On success
    cache.delete(key)
```

### Progressive Delays

```python
import time

def login_with_delay(request):
    ip = get_client_ip(request)
    failures = cache.get(f'failures:{ip}', 0)
    
    # Delay increases with failures
    if failures > 0:
        delay = min(2 ** failures, 60)  # Max 60 seconds
        time.sleep(delay)
    
    # ... authenticate ...
```



## Common Pitfalls

1. **Locking only by IP address** — Attackers use rotating proxies and botnets; lock by both IP and username to catch targeted attacks.
2. **Setting lockout duration too short** — A 1-minute lockout barely slows automated tools; 15-30 minutes is more effective while still being reasonable for legitimate users.

## Best Practices

1. **Lock by IP AND username**: Prevents both distributed and targeted attacks.
2. **Notify users**: Send email on suspicious login attempts.
3. **Use CAPTCHA**: After initial failures, require CAPTCHA.
4. **Monitor and alert**: Track failed login patterns.

## Summary

Protect against brute force with rate limiting, account lockouts, and progressive delays. Use django-axes for a complete solution, and monitor login patterns for attack detection.

## Resources

- [Django Axes](https://django-axes.readthedocs.io/) — Django Axes documentation

---

> 📘 *This lesson is part of the [Django Security Best Practices](https://stanza.dev/courses/django-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*