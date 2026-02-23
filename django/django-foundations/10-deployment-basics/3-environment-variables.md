---
source_course: "django-foundations"
source_lesson: "django-foundations-environment-variables"
---

# Environment Variables and Configuration

## Introduction

Environment variables separate configuration from code, letting you run the same Django application in development, staging, and production with different settings. This is a fundamental principle of the Twelve-Factor App methodology.

## Key Concepts

- **Environment Variable**: A key-value pair set outside your application code, available via `os.environ`.
- **python-dotenv**: A library that loads variables from a `.env` file into the environment.
- **django-environ**: A Django-specific library that parses environment variables with type casting and defaults.

## Real World Context

Every deployment environment has different database credentials, API keys, and feature flags. Hardcoding these values in settings files means you either commit secrets to version control or maintain separate, diverging settings files. Environment variables solve both problems elegantly.

## Deep Dive

### Using os.environ

Python's standard library provides basic access:

```python
import os

# Get with no default (raises KeyError if missing)
SECRET_KEY = os.environ['DJANGO_SECRET_KEY']

# Get with default
DEBUG = os.environ.get('DJANGO_DEBUG', 'False') == 'True'

# Database configuration from environment
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': os.environ.get('DB_NAME', 'mydb'),
        'USER': os.environ.get('DB_USER', 'postgres'),
        'PASSWORD': os.environ.get('DB_PASSWORD', ''),
        'HOST': os.environ.get('DB_HOST', 'localhost'),
        'PORT': os.environ.get('DB_PORT', '5432'),
    }
}
```

### Using python-dotenv

Load environment variables from a `.env` file:

```bash
pip install python-dotenv
```

Create a `.env` file in your project root:

```
# .env (NEVER commit this file!)
DJANGO_SECRET_KEY=your-super-secret-key-here
DJANGO_DEBUG=True
DB_NAME=mydb
DB_USER=postgres
DB_PASSWORD=localpassword
DB_HOST=localhost
DB_PORT=5432
EMAIL_HOST_PASSWORD=smtp-password
```

Load it in settings:

```python
# settings.py
from pathlib import Path
from dotenv import load_dotenv
import os

BASE_DIR = Path(__file__).resolve().parent.parent

# Load .env file
load_dotenv(BASE_DIR / '.env')

SECRET_KEY = os.environ.get('DJANGO_SECRET_KEY')
DEBUG = os.environ.get('DJANGO_DEBUG', 'False') == 'True'
```

### Using django-environ

A more feature-rich alternative with type casting:

```bash
pip install django-environ
```

```python
# settings.py
import environ

env = environ.Env(
    DEBUG=(bool, False),  # Default to False
    ALLOWED_HOSTS=(list, []),
)

# Read .env file
environ.Env.read_env()

DEBUG = env('DEBUG')
SECRET_KEY = env('SECRET_KEY')
ALLOWED_HOSTS = env('ALLOWED_HOSTS')

# Parse database URL
DATABASES = {
    'default': env.db(),  # Reads DATABASE_URL
}
```

With `DATABASE_URL` in `.env`:

```
DATABASE_URL=postgres://user:pass@localhost:5432/mydb
```

### The .gitignore Rule

Always exclude secrets from version control:

```
# .gitignore
.env
*.env
.env.local
.env.production
```

### Providing a Template

Create a `.env.example` (committed to git) as a template:

```
# .env.example
DJANGO_SECRET_KEY=change-me
DJANGO_DEBUG=True
DB_NAME=mydb
DB_USER=postgres
DB_PASSWORD=
DB_HOST=localhost
DB_PORT=5432
```

### Setting Environment Variables in Production

Different deployment platforms have different approaches:

```bash
# Linux systemd service
[Service]
Environment="DJANGO_SECRET_KEY=production-key"
Environment="DJANGO_DEBUG=False"
Environment="DB_NAME=proddb"

# Docker
docker run -e DJANGO_SECRET_KEY=key -e DJANGO_DEBUG=False myapp

# Docker Compose
services:
  web:
    env_file:
      - .env.production
```

### Type Conversion Helpers

Since `os.environ` always returns strings, you need helper functions to convert values to the correct Python types.

```python
import os

def get_bool(name, default=False):
    return os.environ.get(name, str(default)).lower() in ('true', '1', 'yes')

def get_int(name, default=0):
    return int(os.environ.get(name, default))

def get_list(name, default=''):
    value = os.environ.get(name, default)
    return [item.strip() for item in value.split(',') if item.strip()]

# Usage
DEBUG = get_bool('DJANGO_DEBUG')
SESSION_TIMEOUT = get_int('SESSION_TIMEOUT', 3600)
ALLOWED_HOSTS = get_list('ALLOWED_HOSTS', 'localhost,127.0.0.1')
```

## Common Pitfalls

- **Committing `.env` files to version control**: This exposes all your secrets. Add `.env` to `.gitignore` immediately.
- **Forgetting default values**: Without defaults, missing environment variables crash the application on startup. Always provide sensible defaults for non-sensitive settings.
- **Treating all env vars as strings**: `os.environ` returns strings. `os.environ.get('DEBUG')` returns the string `'True'`, not the boolean `True`. Always convert types explicitly.

## Best Practices

- **Provide a `.env.example` file** committed to git so developers know which variables are needed.
- **Use `django-environ` or `python-dotenv`** instead of manually parsing environment variables.
- **Validate required variables at startup**: Fail fast if a critical variable like `SECRET_KEY` is missing rather than crashing at runtime.

## Summary

- Use **environment variables** to separate configuration from code and keep secrets out of version control
- Use `python-dotenv` or `django-environ` to load variables from `.env` files during development
- Never commit `.env` files; provide a `.env.example` template instead
- Convert environment variable strings to proper Python types (bool, int, list) explicitly
- Set environment variables via systemd, Docker, or platform-specific tools in production

## Code Examples

**Loading environment variables from a .env file with python-dotenv for Django settings**

```python
# settings.py
from pathlib import Path
from dotenv import load_dotenv
import os

BASE_DIR = Path(__file__).resolve().parent.parent
load_dotenv(BASE_DIR / '.env')

SECRET_KEY = os.environ.get('DJANGO_SECRET_KEY')
DEBUG = os.environ.get('DJANGO_DEBUG', 'False') == 'True'
ALLOWED_HOSTS = os.environ.get('ALLOWED_HOSTS', 'localhost').split(',')
```


## Resources

- [Django Settings](https://docs.djangoproject.com/en/6.0/topics/settings/) — Official guide to Django settings and configuration

---

> 📘 *This lesson is part of the [Django Foundations](https://stanza.dev/courses/django-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*