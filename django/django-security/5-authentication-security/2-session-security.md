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



## Real World Context

Session hijacking was used in the Firesheep attack, which let anyone on a Wi-Fi network steal Facebook and Twitter sessions in real time. Without Secure and HttpOnly flags on session cookies, any network eavesdropper or XSS vulnerability can compromise user sessions.

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

Django's `login()` function automatically calls `cycle_key()` to rotate the session ID, preventing session fixation attacks. You only need to call `cycle_key()` manually for other authentication state changes (e.g., privilege escalation).

```python
from django.contrib.auth import login

def login_view(request):
    user = authenticate(request, ...)
    if user:
        login(request, user)  # Automatically rotates session ID

# Manual rotation for non-login auth state changes
def elevate_privileges(request):
    request.user.is_admin = True
    request.user.save()
    request.session.cycle_key()  # Rotate after privilege change
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



## Common Pitfalls

1. **Not rotating session IDs on auth state changes** — Django's login() handles rotation automatically, but other privilege changes (e.g., elevating to admin) require a manual cycle_key() call.
2. **Using signed cookie sessions for sensitive data** — Signed cookie sessions are tamper-proof but not encrypted; users can decode and read the session contents.

## Best Practices

1. **Rotate on auth state changes**: Django's login() rotates automatically; call cycle_key() manually for other privilege changes.
2. **Use HTTPS**: Set SESSION_COOKIE_SECURE=True.
3. **Set HttpOnly**: Prevent JavaScript access to session cookie.
4. **Short lifetimes**: Balance security vs user convenience.

## Summary

Secure sessions require HTTPS, HttpOnly cookies, session rotation on login, and appropriate storage backends. Always rotate the session ID after authentication state changes.

## Resources

- [Sessions](https://docs.djangoproject.com/en/6.0/topics/http/sessions/) — Django sessions documentation

---

> 📘 *This lesson is part of the [Django Security Best Practices](https://stanza.dev/courses/django-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*