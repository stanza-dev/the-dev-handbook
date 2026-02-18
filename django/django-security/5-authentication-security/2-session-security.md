---
source_course: "django-security"
source_lesson: "django-security-session-security"
---

# Session Security

## Introduction

Sessions maintain user state across requests. Misconfigured sessions enable session hijacking, fixation attacks, and unauthorized access.

## Key Concepts

**Session Hijacking**: Stealing a session ID to impersonate a user.

**Session Fixation**: Tricking a user into using a session ID chosen by the attacker.

**Session Cookie Attributes**: Secure, HttpOnly, SameSite flags that protect cookies.

## Deep Dive

### Secure Session Configuration

```python
# settings.py
SESSION_ENGINE = 'django.contrib.sessions.backends.db'
SESSION_COOKIE_SECURE = True      # HTTPS only
SESSION_COOKIE_HTTPONLY = True    # No JS access
SESSION_COOKIE_SAMESITE = 'Lax'   # CSRF protection
SESSION_COOKIE_AGE = 86400        # 24 hours
SESSION_EXPIRE_AT_BROWSER_CLOSE = False
SESSION_SAVE_EVERY_REQUEST = True  # Refresh expiry on activity
```

### Preventing Session Fixation

```python
from django.contrib.auth import login

def login_view(request):
    user = authenticate(request, ...)
    if user:
        login(request, user)
        # Rotate session ID after login
        request.session.cycle_key()
```

### Session Storage Options

```python
# Database (default) - most secure
SESSION_ENGINE = 'django.contrib.sessions.backends.db'

# Cache (Redis) - fast, requires secure cache
SESSION_ENGINE = 'django.contrib.sessions.backends.cache'
SESSION_CACHE_ALIAS = 'default'

# Signed cookies - no server storage needed
SESSION_ENGINE = 'django.contrib.sessions.backends.signed_cookies'
```

## Best Practices

1. **Rotate on login**: Call cycle_key() after successful authentication.
2. **Use HTTPS**: Set SESSION_COOKIE_SECURE=True.
3. **Set HttpOnly**: Prevent JavaScript access to session cookie.
4. **Short lifetimes**: Balance security vs user convenience.

## Summary

Secure sessions require HTTPS, HttpOnly cookies, session rotation on login, and appropriate storage backends. Always rotate the session ID after authentication state changes.

## Resources

- [Sessions](https://docs.djangoproject.com/en/6.0/topics/http/sessions/) — Django sessions documentation

---

> 📘 *This lesson is part of the [Django Security Best Practices](https://stanza.dev/courses/django-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*