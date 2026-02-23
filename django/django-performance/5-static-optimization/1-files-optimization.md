---
source_course: "django-performance"
source_lesson: "django-performance-static-files-optimization"
---

# Static Files Optimization

## Introduction

Static files (CSS, JavaScript, images) often account for the majority of page load time. Optimizing their delivery through compression, caching, CDNs, and lazy loading can dramatically improve perceived performance and reduce bandwidth costs.

## Key Concepts

- **WhiteNoise**: A Python package that serves static files directly from your WSGI/ASGI application with compression and caching headers.
- **ManifestStaticFilesStorage**: A Django storage backend that appends content hashes to filenames, enabling aggressive browser caching.
- **CDN (Content Delivery Network)**: A global network of edge servers that cache and serve static files closer to users.
- **django-compressor**: A tool that combines and minifies CSS and JavaScript files to reduce HTTP requests.

## Real World Context

A typical Django page loads 10-20 static files. Without optimization, each requires a separate HTTP request and the browser can't cache them effectively. With WhiteNoise and ManifestStaticFilesStorage, files get content-hashed names (e.g., `main.abc123.css`) allowing 1-year cache headers. Adding a CDN like CloudFront or Cloudflare can reduce latency from 200ms to 20ms for users across the globe.

## Deep Dive

Static file optimization in Django 6 centers on the `STORAGES` configuration dict, compression middleware like WhiteNoise, and far-future cache headers with content-hashed filenames.


## Introduction

Optimizing static file delivery improves page load times significantly.

## WhiteNoise for Production

```python
# pip install whitenoise

# settings.py
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'whitenoise.middleware.WhiteNoiseMiddleware',  # After security
    # ...
]

# Enable compression and caching
# Django 6 uses the STORAGES dict (STATICFILES_STORAGE is removed)
STORAGES = {
    'staticfiles': {
        'BACKEND': 'whitenoise.storage.CompressedManifestStaticFilesStorage',
    },
}

# Django 6 uses the STORAGES dict
STORAGES = {
    'staticfiles': {
        'BACKEND': 'whitenoise.storage.CompressedManifestStaticFilesStorage',
    },
}
```

## Django Compressor

```python
# pip install django-compressor

# settings.py
INSTALLED_APPS = [
    # ...
    'compressor',
]

STATICFILES_FINDERS = [
    'django.contrib.staticfiles.finders.FileSystemFinder',
    'django.contrib.staticfiles.finders.AppDirectoriesFinder',
    'compressor.finders.CompressorFinder',
]

COMPRESS_ENABLED = True
COMPRESS_CSS_FILTERS = [
    'compressor.filters.css_default.CssAbsoluteFilter',
    'compressor.filters.cssmin.rCSSMinFilter',
]
COMPRESS_JS_FILTERS = [
    'compressor.filters.jsmin.JSMinFilter',
]
```

```html
{% load compress %}

{% compress css %}
<link rel="stylesheet" href="{% static 'css/base.css' %}">
<link rel="stylesheet" href="{% static 'css/blog.css' %}">
<link rel="stylesheet" href="{% static 'css/responsive.css' %}">
{% endcompress %}

{% compress js %}
<script src="{% static 'js/jquery.js' %}"></script>
<script src="{% static 'js/app.js' %}"></script>
{% endcompress %}
```

## ManifestStaticFilesStorage

```python
# settings.py
# Django 6 uses the STORAGES dict (STATICFILES_STORAGE is removed)
STORAGES = {
    'staticfiles': {
        'BACKEND': 'django.contrib.staticfiles.storage.ManifestStaticFilesStorage',
    },
}

# Creates versioned file names: style.abc123.css
# Enables aggressive browser caching

# Collect static files
# python manage.py collectstatic
```

## Image Optimization

```python
# pip install django-imagekit

# models.py
from imagekit.models import ImageSpecField
from imagekit.processors import ResizeToFill, ResizeToFit


class Article(models.Model):
    image = models.ImageField(upload_to='articles/')
    
    # Auto-generated thumbnails
    thumbnail = ImageSpecField(
        source='image',
        processors=[ResizeToFill(100, 100)],
        format='JPEG',
        options={'quality': 80}
    )
    
    large = ImageSpecField(
        source='image',
        processors=[ResizeToFit(800, 600)],
        format='JPEG',
        options={'quality': 85}
    )
```

## CDN Integration

```python
# settings.py

# Use CDN for static files
STATIC_URL = 'https://cdn.example.com/static/'

# Or use django-storages with S3/CloudFront
# pip install django-storages boto3

AWS_ACCESS_KEY_ID = 'your-key'
AWS_SECRET_ACCESS_KEY = 'your-secret'
AWS_STORAGE_BUCKET_NAME = 'your-bucket'
AWS_S3_CUSTOM_DOMAIN = f'{AWS_STORAGE_BUCKET_NAME}.s3.amazonaws.com'
AWS_S3_OBJECT_PARAMETERS = {
    'CacheControl': 'max-age=31536000',  # 1 year
}

# DEPRECATED in Django 6: Use STORAGES dict instead
STATICFILES_STORAGE = 'storages.backends.s3boto3.S3Boto3Storage'
STATIC_URL = f'https://{AWS_S3_CUSTOM_DOMAIN}/static/'
```

## Browser Caching Headers

```python
# Nginx configuration
location /static/ {
    alias /var/www/mysite/static/;
    expires 1y;
    add_header Cache-Control "public, immutable";
    add_header Vary Accept-Encoding;
    gzip_static on;
}

# Or in Django middleware
class StaticFileCacheMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
    
    def __call__(self, request):
        response = self.get_response(request)
        
        if request.path.startswith('/static/'):
            response['Cache-Control'] = 'public, max-age=31536000, immutable'
        
        return response
```

## Summary

- The techniques covered in this lesson are essential for production-quality applications.
- Always measure before and after optimizing to verify improvements.
- Start with the simplest approach and add complexity only when needed.

## Common Pitfalls

1. **Serving static files through Django in production** — Django's built-in `runserver` static file handling is extremely slow. Always use WhiteNoise, Nginx, or a CDN.
2. **Not running `collectstatic`** — Forgetting to run `python manage.py collectstatic` before deployment means Nginx/WhiteNoise has no files to serve.
3. **Setting long cache times without content hashing** — If you set `max-age=31536000` but don't use hashed filenames, users will see stale CSS/JS after deployments until the cache expires.

## Best Practices

1. **Use WhiteNoise for simplicity** — It eliminates the need for Nginx to serve static files and works on any platform including Heroku and Docker.
2. **Enable ManifestStaticFilesStorage** — Content-hashed filenames enable aggressive caching while ensuring users always get updated files.
3. **Add a CDN for global reach** — CloudFront, Cloudflare, or similar CDNs cache files at edge locations worldwide, reducing latency significantly.

## Summary

- The techniques covered in this lesson are essential for production-quality Django applications.
- Always measure and profile before optimizing to ensure you're addressing the actual bottleneck.
- Start with the simplest approach and add complexity only when monitoring shows it's needed.

## Code Examples

**WhiteNoise middleware configuration for serving compressed static files**

```python
# Nginx configuration
location /static/ {
    alias /var/www/mysite/static/;
    expires 1y;
    add_header Cache-Control "public, immutable";
    add_header Vary Accept-Encoding;
    gzip_static on;
}

# Or in Django middleware
class StaticFileCacheMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
    
    def __call__(self, request):
        response = self.get_response(request)
        
        if request.path.startswith('/static/'):
            response['Cache-Control'] = 'public, max-age=31536000, immutable'
        
# ...
```


## Resources

- [Static Files](https://docs.djangoproject.com/en/6.0/howto/static-files/deployment/) — Deploying static files

---

> 📘 *This lesson is part of the [Django Performance & Optimization](https://stanza.dev/courses/django-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*