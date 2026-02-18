---
source_course: "django-security"
source_lesson: "django-security-overview"
---

# Django Security Overview

Django provides many built-in protections against common web vulnerabilities.

## Security Checklist

```bash
# Run Django's deployment security check
python manage.py check --deploy
```

## Essential Security Settings

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
CSRF_COOKIE_HTTPONLY = True

# Prevent clickjacking
X_FRAME_OPTIONS = 'DENY'

# Prevent XSS in old browsers
SECURE_BROWSER_XSS_FILTER = True

# Prevent MIME type sniffing
SECURE_CONTENT_TYPE_NOSNIFF = True

# Referrer policy
SECURE_REFERRER_POLICY = 'strict-origin-when-cross-origin'
```

## Security Middleware

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

## Secret Key Generation

```python
# Generate a new secret key
from django.core.management.utils import get_random_secret_key
print(get_random_secret_key())

# Or using Python
import secrets
print(secrets.token_urlsafe(50))
```

## OWASP Top 10 and Django

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

## Keeping Django Updated

```bash
# Check for security updates
pip list --outdated

# Check for known vulnerabilities
pip install safety
safety check

# Or pip-audit
pip install pip-audit
pip-audit
```

## Resources

- [Django Security](https://docs.djangoproject.com/en/6.0/topics/security/) — Official security documentation

---

> 📘 *This lesson is part of the [Django Security Best Practices](https://stanza.dev/courses/django-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*