---
source_course: "django-foundations"
source_lesson: "django-foundations-serving-files-production"
---

# Serving Files in Production

## Introduction

In development, Django conveniently serves both static and media files. In production, this changes completely: Django should never serve files directly. Understanding how to configure file serving for production is essential for a performant, secure deployment.

## Key Concepts

- **collectstatic**: A management command that gathers all static files into a single directory for production serving.
- **WhiteNoise**: A Python library that serves static files directly from your WSGI application.
- **CDN (Content Delivery Network)**: A distributed network of servers that caches and serves static files close to users.

## Real World Context

Serving files through Django in production is slow because every file request goes through the full Python/WSGI stack. A dedicated file server or CDN handles thousands of concurrent file requests efficiently, while Django focuses on dynamic content.

## Deep Dive

### The collectstatic Command

This command copies all static files from your apps and `STATICFILES_DIRS` into `STATIC_ROOT`:

```bash
python manage.py collectstatic
```

Output:

```
128 static files copied to '/path/to/staticfiles'.
```

Configuration:

```python
# settings.py
STATIC_URL = 'static/'
STATIC_ROOT = BASE_DIR / 'staticfiles'  # Where collectstatic copies files
STATICFILES_DIRS = [BASE_DIR / 'static']  # Additional directories to search
```

### File Finders

Django uses finders to locate static files:

```python
STATICFILES_FINDERS = [
    'django.contrib.staticfiles.finders.FileSystemFinder',  # STATICFILES_DIRS
    'django.contrib.staticfiles.finders.AppDirectoriesFinder',  # app/static/
]
```

### WhiteNoise Setup

WhiteNoise serves static files efficiently without a separate web server:

```bash
pip install whitenoise
```

```python
# settings.py
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'whitenoise.middleware.WhiteNoiseMiddleware',  # Right after SecurityMiddleware
    # ...
]

STORAGES = {
    "default": {
        "BACKEND": "django.core.files.storage.FileSystemStorage",
    },
    "staticfiles": {
        "BACKEND": "whitenoise.storage.CompressedManifestStaticFilesStorage",
    },
}
```

WhiteNoise provides:
- **Gzip and Brotli compression** of static files
- **Cache-busting** via content hashes in filenames
- **Forever cache headers** for hashed files

### Serving Media Files in Production

Media files (user uploads) require different handling:

```python
# settings.py
MEDIA_URL = '/media/'
MEDIA_ROOT = BASE_DIR / 'media'
```

With Nginx:

```nginx
server {
    # Serve static files
    location /static/ {
        alias /path/to/staticfiles/;
        expires 30d;
        add_header Cache-Control "public, immutable";
    }

    # Serve media files
    location /media/ {
        alias /path/to/media/;
        expires 7d;
    }

    # Proxy dynamic requests to Django
    location / {
        proxy_pass http://127.0.0.1:8000;
    }
}
```

### Using Cloud Storage

For scalable deployments, use cloud storage backends:

```python
# Using django-storages with AWS S3
pip install django-storages boto3

# settings.py
STORAGES = {
    "default": {
        "BACKEND": "storages.backends.s3boto3.S3Boto3Storage",
        "OPTIONS": {
            "bucket_name": "my-media-bucket",
        },
    },
    "staticfiles": {
        "BACKEND": "storages.backends.s3boto3.S3StaticStorage",
        "OPTIONS": {
            "bucket_name": "my-static-bucket",
        },
    },
}
```

### File Size and Type Validation

Always validate uploaded files to prevent abuse. Django provides a built-in extension validator, and you can write custom validators for file size.

```python
from django.core.validators import FileExtensionValidator
from django.core.exceptions import ValidationError

def validate_file_size(value):
    limit = 5 * 1024 * 1024  # 5 MB
    if value.size > limit:
        raise ValidationError('File too large. Max size is 5 MB.')

class Document(models.Model):
    file = models.FileField(
        upload_to='documents/',
        validators=[
            FileExtensionValidator(allowed_extensions=['pdf', 'doc', 'docx']),
            validate_file_size,
        ]
    )
```

## Common Pitfalls

- **Forgetting to run `collectstatic` during deployment**: Without it, your production server has no static files to serve, resulting in broken CSS and JavaScript.
- **Using Django to serve media files in production**: Never rely on `static()` URL patterns in production. Configure Nginx or a cloud storage backend instead.
- **Not setting cache headers**: Static files with content hashes should have long cache durations. Without proper headers, browsers re-download files unnecessarily.

## Best Practices

- **Use WhiteNoise for simple deployments**: It requires no separate web server configuration for static files.
- **Add file validators** for size and extension on every `FileField` and `ImageField` to prevent abuse.
- **Use cloud storage** (S3, GCS, Azure Blob) for media files in production to separate file storage from your application server.

## Summary

- Run `python manage.py collectstatic` to gather all static files into `STATIC_ROOT` for production
- Use **WhiteNoise** for serving compressed, cache-busted static files without a separate server
- Configure **Nginx** to serve both static and media files directly, bypassing Django
- For scalable deployments, use **cloud storage backends** like AWS S3 via `django-storages`
- Always validate file size and type on `FileField` and `ImageField` with Django validators

## Code Examples

**Configuring WhiteNoise for production static file serving with compression and cache-busting**

```python
# settings.py - Production file serving with WhiteNoise
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'whitenoise.middleware.WhiteNoiseMiddleware',
    # ...
]

STORAGES = {
    "default": {
        "BACKEND": "django.core.files.storage.FileSystemStorage",
    },
    "staticfiles": {
        "BACKEND": "whitenoise.storage.CompressedManifestStaticFilesStorage",
    },
}
```


## Resources

- [Deploying Static Files](https://docs.djangoproject.com/en/6.0/howto/static-files/deployment/) — Official guide to deploying static files in production

---

> 📘 *This lesson is part of the [Django Foundations](https://stanza.dev/courses/django-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*