---
source_course: "django-deployment"
source_lesson: "django-deployment-static-media"
---

# Static and Media Files

## Introduction

In production, Django doesn't serve static files efficiently. You need a proper strategy for serving CSS, JavaScript, images, and user uploads.

## Key Concepts

**Static Files**: CSS, JavaScript, images that ship with your code.

**Media Files**: User-uploaded files stored at runtime.

**WhiteNoise**: Middleware for serving static files from Django.

**CDN**: Content Delivery Network for global distribution.

## Real World Context

This topic directly impacts production application performance. Teams that master these techniques reduce page load times, lower infrastructure costs, and deliver better user experiences.

## Deep Dive

### Configuring Static Files

```python
# settings/production.py
STATIC_URL = '/static/'
STATIC_ROOT = BASE_DIR / 'staticfiles'

# Collect all static files
# python manage.py collectstatic
```

### Using WhiteNoise

```python
# pip install whitenoise

# settings/production.py
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'whitenoise.middleware.WhiteNoiseMiddleware',  # After security
    ...
]

# Django 6 STORAGES dict (replaces STATICFILES_STORAGE)\nSTORAGES = {\n    'staticfiles': {\n        'BACKEND': 'whitenoise.storage.CompressedManifestStaticFilesStorage',\n    },\n}
```

### Media Files with Cloud Storage

```python
# pip install django-storages boto3

# settings/production.py
# Django 6 STORAGES dict (replaces DEFAULT_FILE_STORAGE)\nSTORAGES = {\n    'default': {\n        'BACKEND': 'storages.backends.s3boto3.S3Boto3Storage',\n    },\n    'staticfiles': {\n        'BACKEND': 'whitenoise.storage.CompressedManifestStaticFilesStorage',\n    },\n}
AWS_ACCESS_KEY_ID = os.environ['AWS_ACCESS_KEY_ID']
AWS_SECRET_ACCESS_KEY = os.environ['AWS_SECRET_ACCESS_KEY']
AWS_STORAGE_BUCKET_NAME = 'my-media-bucket'
AWS_S3_REGION_NAME = 'us-east-1'
AWS_DEFAULT_ACL = 'public-read'
```

### CDN Configuration

```python
# Serve static from CDN
STATIC_URL = 'https://cdn.example.com/static/'

# Or use CloudFront with S3
AWS_S3_CUSTOM_DOMAIN = 'd1234567890.cloudfront.net'
```

## Common Pitfalls

1. **Premature optimization** — Always profile before optimizing. Fix the biggest bottleneck first rather than guessing.
2. **Ignoring trade-offs** — Every optimization has costs. Caching adds complexity, indexes slow writes, and async adds cognitive overhead.

## Best Practices

1. **Use WhiteNoise for static**: Simple and efficient.
2. **Cloud storage for media**: S3, GCS, or Azure Blob.
3. **CDN for global distribution**: Faster loading worldwide.
4. **Versioned static files**: Use ManifestStaticFilesStorage.

## Summary

Use WhiteNoise for static files, cloud storage (S3) for media files, and consider a CDN for global distribution. Always run collectstatic before deployment.

## Code Examples

**WhiteNoise and cloud storage configuration for static and media files**

```python
# pip install django-storages boto3

# settings/production.py
# Django 6 STORAGES dict (replaces DEFAULT_FILE_STORAGE)\nSTORAGES = {\n    'default': {\n        'BACKEND': 'storages.backends.s3boto3.S3Boto3Storage',\n    },\n    'staticfiles': {\n        'BACKEND': 'whitenoise.storage.CompressedManifestStaticFilesStorage',\n    },\n}
AWS_ACCESS_KEY_ID = os.environ['AWS_ACCESS_KEY_ID']
AWS_SECRET_ACCESS_KEY = os.environ['AWS_SECRET_ACCESS_KEY']
AWS_STORAGE_BUCKET_NAME = 'my-media-bucket'
AWS_S3_REGION_NAME = 'us-east-1'
AWS_DEFAULT_ACL = 'public-read'
```


## Resources

- [Static Files](https://docs.djangoproject.com/en/6.0/howto/static-files/deployment/) — Deploying static files

---

> 📘 *This lesson is part of the [Django Deployment & Production](https://stanza.dev/courses/django-deployment) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*