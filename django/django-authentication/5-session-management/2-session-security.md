---
source_course: "django-authentication"
source_lesson: "django-authentication-session-security"
---

# Session Security Best Practices

## Introduction

Secure session configuration prevents session hijacking and other attacks.

## Key Concepts

**Session Hijacking**: Stealing session cookies.

**Session Fixation**: Attacker sets victim's session ID.

## Real World Context

A banking application must protect session cookies from theft. If an attacker injects JavaScript via XSS, `SESSION_COOKIE_HTTPONLY = True` prevents the script from reading `document.cookie`. Combined with `SECURE` and `SAMESITE`, this layered defense makes session hijacking significantly harder.

## Deep Dive

### Production Settings

```python
# settings.py
SESSION_COOKIE_SECURE = True    # HTTPS only
SESSION_COOKIE_HTTPONLY = True  # No JavaScript access
SESSION_COOKIE_SAMESITE = 'Lax' # CSRF protection

# Rotate session on login (built-in)
# django.contrib.auth.login() does this automatically
```

### Session Fixation Prevention

```python
from django.contrib.auth import login

def login_view(request):
    # login() automatically rotates session key
    login(request, user)
    # Old session ID is now invalid
```

### Logout Security

```python
from django.contrib.auth import logout

def logout_view(request):
    logout(request)  # Flushes session completely
    return redirect('home')
```

## Common Pitfalls

1. **Forgetting `SESSION_COOKIE_SECURE` in production**: Without it, cookies are sent over plain HTTP. An attacker on the same network (e.g., public Wi-Fi) can intercept them with a packet sniffer.
2. **Setting `SESSION_COOKIE_SAMESITE = 'None'` without `Secure`**: Browsers reject `SameSite=None` cookies that are not also marked `Secure`. This silently breaks sessions.
3. **Not calling `logout()` on session expiry**: When `SESSION_EXPIRE_AT_BROWSER_CLOSE = True`, the cookie is deleted when the browser closes, but the server-side session data remains. Run `clearsessions` periodically to clean up.

## Best Practices

1. **Always use HTTPS**: In production.
2. **Set SameSite**: 'Lax' or 'Strict'.
3. **Short session lifetime**: For sensitive apps.

## Summary

Enable all cookie security flags in production. Django handles session rotation on login automatically.

## Resources

- [Session Security](https://docs.djangoproject.com/en/6.0/topics/http/sessions/#session-security) — Session security

---

> 📘 *This lesson is part of the [Django Authentication & Authorization](https://stanza.dev/courses/django-authentication) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*