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