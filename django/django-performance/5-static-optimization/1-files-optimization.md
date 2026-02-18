---
source_course: "django-performance"
source_lesson: "django-performance-static-files-optimization"
---

# Static Files Optimization

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
STATICFILES_STORAGE = 'whitenoise.storage.CompressedManifestStaticFilesStorage'

# Or for Django 4.2+
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
STATICFILES_STORAGE = 'django.contrib.staticfiles.storage.ManifestStaticFilesStorage'

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

## Resources

- [Static Files](https://docs.djangoproject.com/en/6.0/howto/static-files/deployment/) — Deploying static files

---

> 📘 *This lesson is part of the [Django Performance & Optimization](https://stanza.dev/courses/django-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*