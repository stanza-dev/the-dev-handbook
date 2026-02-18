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