---
source_course: "django-foundations"
source_lesson: "django-foundations-static-files-basics"
---

# Working with Static Files

Static files are assets that don't change per request: CSS stylesheets, JavaScript files, images, and fonts. Django provides a robust system for managing these files.

## Configuring Static Files

Django's `staticfiles` app is included by default:

```python
# mysite/settings.py

INSTALLED_APPS = [
    ...
    'django.contrib.staticfiles',  # Already included
]

# URL prefix for static files
STATIC_URL = 'static/'

# Directory for project-wide static files
STATICFILES_DIRS = [
    BASE_DIR / 'static',
]

# Directory where collectstatic will copy files (production)
STATIC_ROOT = BASE_DIR / 'staticfiles'
```

## Organizing Static Files

### App-level static files

Create a `static` directory in your app:

```
polls/
├── static/
│   └── polls/           # Namespace to avoid conflicts
│       ├── css/
│       │   └── style.css
│       ├── js/
│       │   └── app.js
│       └── images/
│           └── logo.png
├── templates/
└── ...
```

### Project-level static files

```
mysite/
├── static/              # Project-wide static files
│   ├── css/
│   │   └── global.css
│   └── js/
│       └── common.js
├── polls/
└── mysite/
```

## Using Static Files in Templates

To reference a static file in a template, load the `static` template tag library first, then use the `{% static %}` tag to generate the correct URL.

```html
{% load static %}
<!DOCTYPE html>
<html>
<head>
    <link rel="stylesheet" href="{% static 'polls/css/style.css' %}">
</head>
<body>
    <img src="{% static 'polls/images/logo.png' %}" alt="Logo">
    
    <script src="{% static 'polls/js/app.js' %}"></script>
</body>
</html>
```

The `{% static %}` tag generates the full URL to the static file.

## Creating a Stylesheet

Here is an example stylesheet placed in the app's namespaced static directory. This CSS will be available via `{% static 'polls/css/style.css' %}`.

```css
/* polls/static/polls/css/style.css */
body {
    font-family: 'Segoe UI', Tahoma, sans-serif;
    max-width: 800px;
    margin: 0 auto;
    padding: 20px;
}

.question-list {
    list-style: none;
    padding: 0;
}

.question-list li {
    padding: 10px;
    border-bottom: 1px solid #eee;
}

.question-list a {
    color: #092e20;
    text-decoration: none;
}

.question-list a:hover {
    text-decoration: underline;
}
```

## Development vs Production

### Development

Django's development server automatically serves static files when `DEBUG = True`.

### Production

In production, you need to:

1. Set `DEBUG = False`
2. Run `collectstatic` to gather all static files:

```bash
python manage.py collectstatic
```

3. Configure your web server (Nginx, Apache) to serve the files, or use a service like WhiteNoise.

## Using WhiteNoise (Simple Production Setup)

WhiteNoise lets Django serve compressed, cache-busted static files directly without a separate web server. Install it and add it to your middleware.

```bash
pip install whitenoise
```

```python
# settings.py
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'whitenoise.middleware.WhiteNoiseMiddleware',  # Add after SecurityMiddleware
    ...
]

# Enable compression and caching
STORAGES = {
    "default": {
        "BACKEND": "django.core.files.storage.FileSystemStorage",
    },
    "staticfiles": {
        "BACKEND": "whitenoise.storage.CompressedManifestStaticFilesStorage",
    },
}
```

## Common Pitfalls

- **Hardcoding static file paths**: Never write `/static/css/style.css` directly. Always use `{% static 'css/style.css' %}` so paths update when settings change.
- **Forgetting `{% load static %}` in templates**: The `{% static %}` tag is not available by default. You must load it at the top of every template that uses it.
- **Not running `collectstatic` for production**: In production, Django does not serve static files. You must run `python manage.py collectstatic` and configure your web server.

## Best Practices

- **Namespace app static files**: Place files in `app/static/app/` to prevent naming collisions between apps.
- **Use WhiteNoise for simple deployments**: It serves static files directly from your WSGI app without needing a separate web server configuration.
- **Set up `STATICFILES_DIRS`** for project-wide static files that don't belong to any specific app.

## Summary

- Static files (CSS, JS, images) are managed by `django.contrib.staticfiles`
- Configure `STATIC_URL`, `STATICFILES_DIRS`, and `STATIC_ROOT` in settings
- Use `{% load static %}` and `{% static 'path/to/file' %}` in templates
- In development, Django serves static files automatically when `DEBUG = True`
- In production, run `collectstatic` and use WhiteNoise or a web server like Nginx

## Code Examples

**Configuring static file settings and referencing static files in templates**

```python
# settings.py
STATIC_URL = 'static/'
STATICFILES_DIRS = [BASE_DIR / 'static']
STATIC_ROOT = BASE_DIR / 'staticfiles'

# In templates:
# {% load static %}
# <link rel="stylesheet" href="{% static 'css/style.css' %}">
```


## Resources

- [Managing Static Files](https://docs.djangoproject.com/en/6.0/howto/static-files/) — Official guide to static files in Django

---

> 📘 *This lesson is part of the [Django Foundations](https://stanza.dev/courses/django-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*