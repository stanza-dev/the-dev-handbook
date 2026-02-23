---
source_course: "django-security"
source_lesson: "django-security-overview"
---

# Django Security Overview

## Introduction

Django is one of the most security-conscious web frameworks available, providing built-in protections against the most common web vulnerabilities. This lesson covers Django's security architecture, essential production settings, and the tools you need to audit your deployment.

## Key Concepts

- **Defense in Depth**: Django layers multiple protections so a single failure doesn't compromise the whole system.
- **Secure by Default**: Most dangerous features (DEBUG, wildcard ALLOWED_HOSTS) must be explicitly enabled.
- **manage.py check --deploy**: Built-in command that audits your settings against known security risks.
- **OWASP Top 10**: Industry-standard list of the most critical web application security risks.

## Real World Context

Misconfigured Django deployments are routinely discovered through automated scanners. A single forgotten DEBUG=True in production leaks your entire settings module, database credentials, and source code to anyone who triggers an error. Running `check --deploy` before every release catches these issues before attackers do.

## Deep Dive

### Security Checklist

```bash
# Run Django's deployment security check
python manage.py check --deploy
```

### Essential Security Settings

```python
# settings/production.py

# NEVER run with DEBUG=True in production
DEBUG = False

# Secret key - use environment variable, never commit
SECRET_KEY = os.environ['DJANGO_SECRET_KEY']

# Only allow your domain(s)
ALLOWED_HOSTS = ['example.com', 'www.example.com']

# Force HTTPS
SECURE_SSL_REDIRECT = True

# HSTS - Tell browsers to only use HTTPS
SECURE_HSTS_SECONDS = 31536000  # 1 year
SECURE_HSTS_INCLUDE_SUBDOMAINS = True
SECURE_HSTS_PRELOAD = True

# Secure cookies
SESSION_COOKIE_SECURE = True
CSRF_COOKIE_SECURE = True
SESSION_COOKIE_HTTPONLY = True
CSRF_COOKIE_HTTPONLY = False  # Keep False if using cookie-based AJAX CSRF pattern

# Prevent clickjacking
X_FRAME_OPTIONS = 'DENY'

# Note: SECURE_BROWSER_XSS_FILTER was removed in Django 4.0
# Modern protection uses Content Security Policy instead (see SECURE_CSP)
# SECURE_CSP is configured via django.middleware.csp.ContentSecurityPolicyMiddleware

# Prevent MIME type sniffing
SECURE_CONTENT_TYPE_NOSNIFF = True

# Referrer policy
SECURE_REFERRER_POLICY = 'strict-origin-when-cross-origin'
```

### Security Middleware

```python
# Ensure security middleware is enabled
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',  # First!
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]
```

### Secret Key Generation

```python
# Generate a new secret key
from django.core.management.utils import get_random_secret_key
print(get_random_secret_key())

# Or using Python
import secrets
print(secrets.token_urlsafe(50))
```

### OWASP Top 10 and Django

Django provides protection against many OWASP Top 10 vulnerabilities:

1. **Injection** - ORM prevents SQL injection
2. **Broken Authentication** - Built-in auth with secure defaults
3. **Sensitive Data Exposure** - HTTPS enforcement, secure cookies
4. **XML External Entities** - Not applicable (Django doesn't parse XML by default)
5. **Broken Access Control** - Permission system, decorators
6. **Security Misconfiguration** - manage.py check --deploy
7. **XSS** - Template auto-escaping
8. **Insecure Deserialization** - Signed cookies, careful with pickle
9. **Using Components with Known Vulnerabilities** - Keep Django updated
10. **Insufficient Logging** - Django logging framework

### Keeping Django Updated

```bash
# Check for security updates
pip list --outdated

# Check for known vulnerabilities
pip install safety
safety scan

# Or pip-audit
pip install pip-audit
pip-audit
```

## Common Pitfalls

1. **Leaving DEBUG=True in production** — Exposes stack traces, settings, and SQL queries to anyone who triggers an error.
2. **Forgetting to run check --deploy** — Many misconfigurations only surface under this flag, not during normal development.
3. **Ordering middleware incorrectly** — SecurityMiddleware must be first; placing it later means earlier middleware operates without its protections.

## Best Practices

1. **Automate check --deploy in CI/CD** — Fail the build if security checks don't pass.
2. **Use environment variables for secrets** — Never hardcode SECRET_KEY or database credentials.
3. **Keep Django and dependencies updated** — Subscribe to Django's security mailing list for patch notifications.

## Summary

- Django ships with strong defaults, but production requires explicit configuration of HTTPS, cookies, and host validation.
- Run `manage.py check --deploy` before every release to catch misconfigurations.
- Layer your protections: middleware ordering, secure cookies, HSTS, and CSP all work together.

## Code Examples

**Essential Django production security settings that should be configured before any deployment**

```python
# settings/production.py
import os

DEBUG = False
SECRET_KEY = os.environ['DJANGO_SECRET_KEY']
ALLOWED_HOSTS = ['example.com', 'www.example.com']

SECURE_SSL_REDIRECT = True
SECURE_HSTS_SECONDS = 31536000
SECURE_HSTS_INCLUDE_SUBDOMAINS = True
SECURE_HSTS_PRELOAD = True

SESSION_COOKIE_SECURE = True
CSRF_COOKIE_SECURE = True
SESSION_COOKIE_HTTPONLY = True
SECURE_CONTENT_TYPE_NOSNIFF = True
X_FRAME_OPTIONS = 'DENY'
```


## Resources

- [Django Security](https://docs.djangoproject.com/en/6.0/topics/security/) — Official security documentation

---

> 📘 *This lesson is part of the [Django Security Best Practices](https://stanza.dev/courses/django-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*