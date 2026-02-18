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

## Resources

- [Deployment Checklist](https://docs.djangoproject.com/en/6.0/howto/deployment/checklist/) — Official deployment checklist for Django

---

> 📘 *This lesson is part of the [Django Foundations](https://stanza.dev/courses/django-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*