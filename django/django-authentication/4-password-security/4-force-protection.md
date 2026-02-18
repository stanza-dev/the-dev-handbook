---
source_course: "django-authentication"
source_lesson: "django-authentication-brute-force-protection"
---

# Brute Force Protection

## Introduction

Protect your login form from brute force attacks with rate limiting and account lockout.

## Key Concepts

**Rate Limiting**: Limit login attempts.

**Account Lockout**: Temporarily disable accounts.

## Deep Dive

### Using django-axes

```python
# pip install django-axes

# settings.py
INSTALLED_APPS = ['axes', ...]

MIDDLEWARE = [
    'axes.middleware.AxesMiddleware',
    ...
]

AUTHENTICATION_BACKENDS = [
    'axes.backends.AxesStandaloneBackend',
    'django.contrib.auth.backends.ModelBackend',
]

# Configuration
AXES_FAILURE_LIMIT = 5  # Lock after 5 failures
AXES_COOLOFF_TIME = timedelta(minutes=15)  # Lock duration
AXES_LOCKOUT_CALLABLE = 'myapp.lockout.lockout_response'
```

### Custom Rate Limiting

```python
from django.core.cache import cache

def check_rate_limit(request, username):
    key = f'login_attempts:{username}'
    attempts = cache.get(key, 0)
    
    if attempts >= 5:
        return False  # Locked out
    
    cache.set(key, attempts + 1, timeout=900)  # 15 min
    return True
```

## Best Practices

1. **Lock by IP and username**: Prevent distributed attacks.
2. **Use exponential backoff**: Increase lockout time.
3. **Log failed attempts**: For security monitoring.

## Summary

Use django-axes for production. Configure reasonable limits. Log and monitor failed attempts.

## Resources

- [django-axes](https://django-axes.readthedocs.io/) — Brute force protection

---

> 📘 *This lesson is part of the [Django Authentication & Authorization](https://stanza.dev/courses/django-authentication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*