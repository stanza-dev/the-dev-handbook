---
source_course: "django-security"
source_lesson: "django-security-csrf-configuration"
---

# CSRF Cookie Configuration

## Introduction

CSRF protection depends on cookies and tokens working together. Proper cookie configuration ensures protection works correctly while supporting legitimate cross-origin requests.

## Key Concepts

**SameSite Cookie Attribute**: Controls when cookies are sent with cross-origin requests.

**CSRF_TRUSTED_ORIGINS**: Domains allowed to make cross-origin POST requests.

**Double Submit Cookie**: Pattern where token in cookie must match token in request.

## Deep Dive

### Cookie Configuration

```python
# settings.py

# Secure: Only send over HTTPS
CSRF_COOKIE_SECURE = True

# HttpOnly: Prevent JavaScript access (use header instead)
CSRF_COOKIE_HTTPONLY = True

# SameSite: Control cross-origin behavior
CSRF_COOKIE_SAMESITE = 'Lax'  # or 'Strict'

# Domain: Share across subdomains
CSRF_COOKIE_DOMAIN = '.example.com'
```

### Trusted Origins (Django 4.0+)

```python
# Required for cross-origin HTTPS requests
CSRF_TRUSTED_ORIGINS = [
    'https://example.com',
    'https://www.example.com',
    'https://app.example.com',
]
```

### SameSite Values Explained

```
Strict: Never send cookie cross-origin (breaks login from external links)
Lax: Send on navigation, not on POST (good default)
None: Always send (requires Secure=True)
```

## Best Practices

1. **Use Lax for SameSite**: Balances security and usability.
2. **Always HTTPS in production**: Set CSRF_COOKIE_SECURE=True.
3. **Configure TRUSTED_ORIGINS**: Required for cross-origin POST.

## Summary

CSRF cookie settings control security vs compatibility tradeoffs. Use Secure=True in production, SameSite=Lax as a good default, and configure CSRF_TRUSTED_ORIGINS for any cross-origin POST requirements.

## Resources

- [CSRF Settings](https://docs.djangoproject.com/en/6.0/ref/settings/#csrf-cookie-samesite) — CSRF cookie settings reference

---

> 📘 *This lesson is part of the [Django Security Best Practices](https://stanza.dev/courses/django-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*