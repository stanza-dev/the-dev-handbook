---
source_course: "django-deployment"
source_lesson: "django-deployment-production-settings"
---

# Production Settings

## Introduction

Properly configuring Django for production is critical for both security and performance. Production settings differ significantly from development: DEBUG must be off, secrets must come from environment variables, and security headers must be enabled to protect against common attacks.

## Key Concepts

- **Settings Module Splitting**: Separating settings into base, development, and production files to avoid accidentally running development config in production.
- **Environment Variables**: External configuration values (secrets, database URLs) that keep sensitive data out of source code.
- **Security Headers**: HTTP response headers like HSTS, X-Frame-Options, and Content-Type-Options that protect against common web attacks.
- **Deployment Check**: Django's `manage.py check --deploy` command that audits your settings for production readiness.

## Real World Context

Every Django deployment that exposes DEBUG=True in production risks leaking database credentials, secret keys, and internal file paths to attackers through error pages. In 2023, multiple high-profile data breaches were traced to exposed Django debug pages. The `check --deploy` command catches these issues before they reach production.

## Deep Dive

Production settings must prioritize security and performance over developer convenience. The most critical change is `DEBUG = False`, which disables detailed error pages and enables security checks.


## Introduction

Properly configuring Django for production is critical for security and performance.

## Settings Structure

```python
myproject/
├── settings/
│   ├── __init__.py
│   ├── base.py       # Shared settings
│   ├── development.py
│   └── production.py
```

```python
# settings/base.py
from pathlib import Path
import os

BASE_DIR = Path(__file__).resolve().parent.parent.parent

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    # Your apps
    'blog',
    'accounts',
]

# ... other shared settings
```

```python
# settings/production.py
from .base import *
import os

DEBUG = False

SECRET_KEY = os.environ['DJANGO_SECRET_KEY']

ALLOWED_HOSTS = os.environ.get('ALLOWED_HOSTS', '').split(',')

# Database
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': os.environ['DB_NAME'],
        'USER': os.environ['DB_USER'],
        'PASSWORD': os.environ['DB_PASSWORD'],
        'HOST': os.environ['DB_HOST'],
        'PORT': os.environ.get('DB_PORT', '5432'),
        'CONN_MAX_AGE': 60,
        'OPTIONS': {
            'sslmode': 'require',
        },
    }
}

# Security settings
SECURE_SSL_REDIRECT = True
SESSION_COOKIE_SECURE = True
CSRF_COOKIE_SECURE = True
# Note: SECURE_BROWSER_XSS_FILTER was removed in Django 3.0.
# Use Content-Security-Policy header instead for modern XSS protection.
SECURE_CONTENT_TYPE_NOSNIFF = True
SECURE_HSTS_SECONDS = 31536000
SECURE_HSTS_INCLUDE_SUBDOMAINS = True
SECURE_HSTS_PRELOAD = True

# Static files
STATIC_ROOT = BASE_DIR / 'staticfiles'
# Django 6 uses STORAGES dict instead of STATICFILES_STORAGE\nSTORAGES = {\n    'staticfiles': {\n        'BACKEND': 'whitenoise.storage.CompressedManifestStaticFilesStorage',\n    },\n}

# Logging
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'formatters': {
        'verbose': {
            'format': '{levelname} {asctime} {module} {message}',
            'style': '{',
        },
    },
    'handlers': {
        'console': {
            'class': 'logging.StreamHandler',
            'formatter': 'verbose',
        },
    },
    'root': {
        'handlers': ['console'],
        'level': 'INFO',
    },
    'loggers': {
        'django': {
            'handlers': ['console'],
            'level': 'INFO',
            'propagate': False,
        },
    },
}

# Email
EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'
EMAIL_HOST = os.environ.get('EMAIL_HOST', 'smtp.sendgrid.net')
EMAIL_PORT = 587
EMAIL_USE_TLS = True
EMAIL_HOST_USER = os.environ.get('EMAIL_HOST_USER')
EMAIL_HOST_PASSWORD = os.environ.get('EMAIL_HOST_PASSWORD')
```

## Environment Variables

```python
# Using python-dotenv
# pip install python-dotenv

# settings/base.py
from dotenv import load_dotenv
load_dotenv()

# Or using django-environ
# pip install django-environ

import environ

env = environ.Env(
    DEBUG=(bool, False),
    ALLOWED_HOSTS=(list, []),
)

DEBUG = env('DEBUG')
SECRET_KEY = env('SECRET_KEY')
DATABASES = {'default': env.db()}
```

```bash
# .env file (never commit this!)
DJANGO_SECRET_KEY=your-super-secret-key-here
DEBUG=False
ALLOWED_HOSTS=example.com,www.example.com
DATABASE_URL=postgres://user:pass@host:5432/dbname
```

## Pre-Deployment Checklist

```bash
# Run deployment check
python manage.py check --deploy

# Example output:
# System check identified some issues:
# WARNINGS:
# ?: (security.W004) You have not set a value for the SECURE_HSTS_SECONDS setting.
# ?: (security.W008) Your SECURE_SSL_REDIRECT setting is not set to True.
```

```python
# Addressing common warnings

# 1. SECURE_SSL_REDIRECT - Redirect HTTP to HTTPS
SECURE_SSL_REDIRECT = True

# 2. SECURE_HSTS_SECONDS - HTTP Strict Transport Security
SECURE_HSTS_SECONDS = 31536000  # 1 year

# 3. SESSION_COOKIE_SECURE - Send cookies only over HTTPS
SESSION_COOKIE_SECURE = True
CSRF_COOKIE_SECURE = True

# 4. X-Frame-Options
X_FRAME_OPTIONS = 'DENY'

# 5. Secret key must be set and unique
# Never use the default or commit to version control
```

## Summary

- The techniques covered in this lesson are essential for production-quality applications.
- Always measure before and after optimizing to verify improvements.
- Start with the simplest approach and add complexity only when needed.

## Common Pitfalls

1. **Committing secrets to version control** — Never put SECRET_KEY, database passwords, or API keys in settings files. Use environment variables or a secrets manager.
2. **Forgetting ALLOWED_HOSTS** — With DEBUG=False, Django returns 400 errors if the request's Host header doesn't match ALLOWED_HOSTS. Always configure this.
3. **Using the same SECRET_KEY across environments** — Each environment (dev, staging, production) should have its own unique SECRET_KEY.



> **ASGI Caveat (Django 6 docs)**: When using ASGI servers (Uvicorn, Daphne), persistent database connections must be disabled (`CONN_MAX_AGE = 0`). Use your database backend's built-in connection pooling (e.g., PostgreSQL `"pool": True` option, available since Django 5.1) or an external pooler like PgBouncer instead.

## Best Practices

1. **Run `check --deploy` in CI** — Add `python manage.py check --deploy` to your CI pipeline to catch security misconfigurations automatically.
2. **Use django-environ for environment variables** — It provides type casting, defaults, and database URL parsing in a clean API.
3. **Enable all security headers** — HSTS, secure cookies, SSL redirect, and content type nosniff should all be enabled for production.

## Summary

- The techniques covered in this lesson are essential for production-quality Django applications.
- Always measure and profile before optimizing to ensure you're addressing the actual bottleneck.
- Start with the simplest approach and add complexity only when monitoring shows it's needed.

## Code Examples

**Production settings with security headers, environment variables, and STORAGES configuration**

```python
# settings/production.py
from .base import *
import os

DEBUG = False

SECRET_KEY = os.environ['DJANGO_SECRET_KEY']

ALLOWED_HOSTS = os.environ.get('ALLOWED_HOSTS', '').split(',')

# Database
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': os.environ['DB_NAME'],
        'USER': os.environ['DB_USER'],
        'PASSWORD': os.environ['DB_PASSWORD'],
        'HOST': os.environ['DB_HOST'],
        'PORT': os.environ.get('DB_PORT', '5432'),
        'CONN_MAX_AGE': 60,
# ...
```


## Resources

- [Deployment Checklist](https://docs.djangoproject.com/en/6.0/howto/deployment/checklist/) — Official deployment checklist

---

> 📘 *This lesson is part of the [Django Deployment & Production](https://stanza.dev/courses/django-deployment) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*