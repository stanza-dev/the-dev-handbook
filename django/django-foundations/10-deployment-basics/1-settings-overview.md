---
source_course: "django-foundations"
source_lesson: "django-foundations-production-settings-overview"
---

# Production Settings

Deploying Django to production requires careful configuration to ensure security and performance.

## Key Differences: Development vs Production

| Development | Production |
|-------------|------------|
| `DEBUG = True` | `DEBUG = False` |
| SQLite database | PostgreSQL/MySQL |
| Django dev server | Gunicorn/uWSGI |
| Static served by Django | Static served by web server/CDN |
| Errors shown in browser | Errors logged, generic 500 page |

## Critical Production Settings

These are the minimum settings you must change when deploying to production. Never use development defaults in a live environment.

```python
# settings.py

import os

# SECURITY: Never run DEBUG=True in production!
DEBUG = False

# SECURITY: Define allowed hosts
ALLOWED_HOSTS = ['yourdomain.com', 'www.yourdomain.com']

# SECURITY: Keep secret key secret!
SECRET_KEY = os.environ.get('DJANGO_SECRET_KEY')

# Use environment variables for sensitive data
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': os.environ.get('DB_NAME'),
        'USER': os.environ.get('DB_USER'),
        'PASSWORD': os.environ.get('DB_PASSWORD'),
        'HOST': os.environ.get('DB_HOST', 'localhost'),
        'PORT': os.environ.get('DB_PORT', '5432'),
    }
}
```

## Security Settings

Django provides several settings to enforce HTTPS, prevent clickjacking, and add content security policies.

```python
# HTTPS Settings
SECURE_SSL_REDIRECT = True  # Redirect HTTP to HTTPS
SESSION_COOKIE_SECURE = True  # Only send session cookie over HTTPS
CSRF_COOKIE_SECURE = True  # Only send CSRF cookie over HTTPS

# HTTP Strict Transport Security
SECURE_HSTS_SECONDS = 31536000  # 1 year
SECURE_HSTS_INCLUDE_SUBDOMAINS = True
SECURE_HSTS_PRELOAD = True

# Content Security (new in Django 6.0)
from django.utils.csp import CSP

SECURE_CSP = {
    'default-src': [CSP.SELF],
    'script-src': [CSP.SELF],
    'style-src': [CSP.SELF],
}

# Important: You must add ContentSecurityPolicyMiddleware to MIDDLEWARE:
# MIDDLEWARE = [
#     ...
#     'django.middleware.security.ContentSecurityPolicyMiddleware',
#     ...
# ]

# Other security settings
SECURE_CONTENT_TYPE_NOSNIFF = True
X_FRAME_OPTIONS = 'DENY'
```

## Separating Settings

Organize settings for different environments:

```
mysite/
├── settings/
│   ├── __init__.py
│   ├── base.py       # Shared settings
│   ├── development.py
│   └── production.py
```

```python
# settings/base.py
from pathlib import Path

BASE_DIR = Path(__file__).resolve().parent.parent.parent

INSTALLED_APPS = [...]
MIDDLEWARE = [...]
# ... common settings
```

```python
# settings/development.py
from .base import *

DEBUG = True
ALLOWED_HOSTS = ['localhost', '127.0.0.1']

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
    }
}
```

```python
# settings/production.py
from .base import *
import os

DEBUG = False
ALLOWED_HOSTS = [os.environ.get('DOMAIN')]

# Production database
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': os.environ.get('DB_NAME'),
        'USER': os.environ.get('DB_USER'),
        'PASSWORD': os.environ.get('DB_PASSWORD'),
        'HOST': os.environ.get('DB_HOST'),
        'PORT': '5432',
    }
}

# Security
SECURE_SSL_REDIRECT = True
SESSION_COOKIE_SECURE = True
CSRF_COOKIE_SECURE = True
```

Run with specific settings:

```bash
# Development
export DJANGO_SETTINGS_MODULE=mysite.settings.development

# Production
export DJANGO_SETTINGS_MODULE=mysite.settings.production
```

## Environment Variables

Use a `.env` file with python-dotenv:

```bash
pip install python-dotenv
```

```
# .env (never commit this file!)
DJANGO_SECRET_KEY=your-secret-key-here
DB_NAME=mydb
DB_USER=dbuser
DB_PASSWORD=dbpassword
DB_HOST=localhost
```

```python
# settings/base.py
from dotenv import load_dotenv
load_dotenv()

SECRET_KEY = os.environ.get('DJANGO_SECRET_KEY')
```

## Common Pitfalls

- **Leaving `DEBUG = True` in production**: This exposes source code, settings, and local variables in error pages to anyone visiting your site.
- **Committing `SECRET_KEY` to version control**: The secret key is used for cryptographic signing. If compromised, attackers can forge cookies and tokens.
- **Setting `ALLOWED_HOSTS = ['*']`**: This defeats the purpose of Host header validation. Always list your specific domains.

## Best Practices

- **Use environment variables** for all secrets (`SECRET_KEY`, database credentials, API keys).
- **Split settings into multiple files**: Use `base.py`, `development.py`, and `production.py` to keep environment-specific settings separate.
- **Run Django's deployment checklist**: Execute `python manage.py check --deploy` to identify security issues before going live.

## Summary

- Set `DEBUG = False` and configure `ALLOWED_HOSTS` with your actual domains
- Use **environment variables** for `SECRET_KEY`, database credentials, and other secrets
- Enable security settings: `SECURE_SSL_REDIRECT`, `SESSION_COOKIE_SECURE`, `CSRF_COOKIE_SECURE`, HSTS
- Split settings into `base.py`, `development.py`, and `production.py` for clean environment management
- Run `python manage.py check --deploy` to verify production readiness

## Code Examples

**Essential Django production settings for security and deployment**

```python
import os

# Production settings
DEBUG = False
ALLOWED_HOSTS = ['yourdomain.com', 'www.yourdomain.com']
SECRET_KEY = os.environ.get('DJANGO_SECRET_KEY')

# Security
SECURE_SSL_REDIRECT = True
SESSION_COOKIE_SECURE = True
CSRF_COOKIE_SECURE = True
SECURE_HSTS_SECONDS = 31536000
```


## Resources

- [Deployment Checklist](https://docs.djangoproject.com/en/6.0/howto/deployment/checklist/) — Official deployment checklist for Django

---

> 📘 *This lesson is part of the [Django Foundations](https://stanza.dev/courses/django-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*