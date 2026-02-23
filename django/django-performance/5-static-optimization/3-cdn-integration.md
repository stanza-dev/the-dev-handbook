---
source_course: "django-performance"
source_lesson: "django-performance-cdn-integration"
---

# CDN Integration

## Introduction

Content Delivery Networks cache static files at edge locations worldwide, reducing latency for global users.

## Key Concepts

- **CDN (Content Delivery Network)**: A globally distributed network of servers that caches and serves static files from locations geographically close to users.
- **django-storages**: A third-party package providing storage backends for cloud services like AWS S3, Google Cloud Storage, and Azure Blob Storage.
- **STORAGES dict**: Django 6's configuration setting for file storage backends, replacing the deprecated `STATICFILES_STORAGE` and `DEFAULT_FILE_STORAGE` settings.
- **Cache-Control headers**: HTTP headers that tell browsers and CDNs how long to cache a file before requesting a fresh copy.

## Real World Context

This topic directly impacts production application performance. Teams that master these techniques reduce page load times, lower infrastructure costs, and deliver better user experiences.

## Deep Dive

### Basic CDN Setup

```python
# settings.py
STATIC_URL = 'https://cdn.example.com/static/'
MEDIA_URL = 'https://cdn.example.com/media/'
```

### AWS CloudFront + S3

```python
# pip install django-storages boto3

AWS_ACCESS_KEY_ID = os.environ['AWS_ACCESS_KEY_ID']
AWS_SECRET_ACCESS_KEY = os.environ['AWS_SECRET_ACCESS_KEY']
AWS_STORAGE_BUCKET_NAME = 'my-bucket'
AWS_S3_REGION_NAME = 'us-east-1'
AWS_S3_CUSTOM_DOMAIN = 'd1234.cloudfront.net'

AWS_S3_OBJECT_PARAMETERS = {
    'CacheControl': 'max-age=31536000',  # 1 year
}

# Separate storage for static and media
# Django 6 STORAGES dict
STORAGES = {
    'default': {
        'BACKEND': 'myapp.storage.MediaStorage',
    },
    'staticfiles': {
        'BACKEND': 'myapp.storage.StaticStorage',
    },
}
# (included in STORAGES dict above)
```

```python
# storage.py
from storages.backends.s3boto3 import S3Boto3Storage

class StaticStorage(S3Boto3Storage):
    location = 'static'
    default_acl = 'public-read'

class MediaStorage(S3Boto3Storage):
    location = 'media'
    file_overwrite = False
```

### Cache Headers

```python
# Immutable assets (hashed names)
AWS_S3_OBJECT_PARAMETERS = {
    'CacheControl': 'max-age=31536000, immutable',
}

# Mutable assets
class MediaStorage(S3Boto3Storage):
    object_parameters = {
        'CacheControl': 'max-age=86400',  # 1 day
    }
```

## Common Pitfalls

1. **Premature optimization** — Always profile before optimizing. Fix the biggest bottleneck first rather than guessing.
2. **Ignoring trade-offs** — Every optimization has costs. Caching adds complexity, indexes slow writes, and async adds cognitive overhead.

## Best Practices

1. **Use long cache times**: With hashed file names.
2. **Separate static and media**: Different cache strategies.
3. **Enable compression**: CloudFront/CDN-level gzip.

## Summary

Use CDN for global distribution of static files. Configure long cache times with hashed names. Use django-storages for S3/CloudFront integration.

## Code Examples

**AWS S3 configuration with Django 6 STORAGES dict**

```python
# Django 6 STORAGES dict (replaces STATICFILES_STORAGE and DEFAULT_FILE_STORAGE)
STORAGES = {
    'default': {
        'BACKEND': 'storages.backends.s3boto3.S3Boto3Storage',
    },
    'staticfiles': {
        'BACKEND': 'storages.backends.s3boto3.S3Boto3Storage',
    },
}

# S3 configuration
AWS_STORAGE_BUCKET_NAME = 'my-bucket'
AWS_S3_REGION_NAME = 'us-east-1'
AWS_S3_CUSTOM_DOMAIN = f'{AWS_STORAGE_BUCKET_NAME}.s3.amazonaws.com'
AWS_S3_OBJECT_PARAMETERS = {
    'CacheControl': 'max-age=86400',
}
```


## Resources

- [django-storages](https://django-storages.readthedocs.io/) — Django storages documentation

---

> 📘 *This lesson is part of the [Django Performance & Optimization](https://stanza.dev/courses/django-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*