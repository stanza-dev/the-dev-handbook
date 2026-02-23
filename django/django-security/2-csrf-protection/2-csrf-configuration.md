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



## Real World Context

Misconfigured CSRF cookies are a common source of 403 errors after deploying to HTTPS. Forgetting CSRF_TRUSTED_ORIGINS breaks cross-origin form submissions entirely, leaving developers scrambling to debug production issues that never appeared in local development.

## Deep Dive

### Cookie Configuration

```python
# settings.py

# Secure: Only send over HTTPS
CSRF_COOKIE_SECURE = True

# HttpOnly: Set False if JavaScript needs to read the CSRF cookie
CSRF_COOKIE_HTTPONLY = False

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



## Common Pitfalls

1. **Setting CSRF_COOKIE_HTTPONLY=True with JavaScript CSRF reads** — If your frontend reads the CSRF token from the cookie (common in SPAs), HttpOnly blocks that access entirely.
2. **Forgetting CSRF_TRUSTED_ORIGINS after moving to HTTPS** — Django 4.0+ requires explicit trusted origins for cross-origin HTTPS requests; omitting this causes silent 403 errors.

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