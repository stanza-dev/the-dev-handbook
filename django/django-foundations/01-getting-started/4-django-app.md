---
source_course: "django-foundations"
source_lesson: "django-foundations-creating-django-app"
---

# Creating Your First Django App

In Django, a **project** is a collection of configurations and **apps**. An app is a web application that does something—a blog, a poll system, a user authentication system. A project can contain multiple apps.

## Projects vs Apps

```
mysite/                 # PROJECT - configuration container
├── manage.py
├── mysite/             # Project settings
│   ├── settings.py
│   └── urls.py
├── blog/               # APP - does something specific
│   ├── models.py
│   ├── views.py
│   └── ...
└── polls/              # Another APP
    ├── models.py
    └── ...
```

## Creating an App

Let's create a simple polls application:

```bash
python manage.py startapp polls
```

This creates:

```
polls/
├── __init__.py
├── admin.py          # Admin interface configuration
├── apps.py           # App configuration
├── migrations/       # Database migrations
│   └── __init__.py
├── models.py         # Data models
├── tests.py          # Tests
└── views.py          # View functions
```

## Register the App

Tell Django about your new app in `settings.py`:

```python
# mysite/settings.py

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'polls',  # Add your app here
]
```

## Creating Your First View

Views handle HTTP requests and return responses. Let's create a simple view:

```python
# polls/views.py
from django.http import HttpResponse


def index(request):
    return HttpResponse("Hello, world! This is the polls index.")
```

## Mapping URLs to Views

Create a `urls.py` file in your app:

```python
# polls/urls.py
from django.urls import path

from . import views

urlpatterns = [
    path('', views.index, name='index'),
]
```

Include the app URLs in the project's main `urls.py`:

```python
# mysite/urls.py
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path('polls/', include('polls.urls')),
    path('admin/', admin.site.urls),
]
```

## Test Your App

Start the server and visit `http://127.0.0.1:8000/polls/`:

```bash
python manage.py runserver
```

You should see "Hello, world! This is the polls index."

## Understanding the Request Flow

1. User visits `/polls/`
2. Django looks at `mysite/urls.py`
3. Matches `polls/` → includes `polls/urls.py`
4. Matches empty string `''` → calls `views.index`
5. `index()` receives the request and returns a response
6. Browser displays the response

```
http://127.0.0.1:8000/polls/
          │
          ▼
    mysite/urls.py
    path('polls/', include('polls.urls'))
          │
          ▼
    polls/urls.py
    path('', views.index)
          │
          ▼
    polls/views.py
    def index(request): return HttpResponse(...)
```

## Common Pitfalls

- **Forgetting to register the app in INSTALLED_APPS**: Your app's models, templates, and static files won't be discovered until you add it to `settings.py`.
- **Not creating `urls.py` in the app**: Django does not auto-create this file. You must create it manually and include it in the project's `urls.py`.
- **Missing the trailing slash in URL patterns**: Django expects `path('polls/', include('polls.urls'))` with a trailing slash by default.

## Best Practices

- **Keep apps focused**: Each app should handle one specific piece of functionality (e.g., polls, blog, accounts).
- **Use `include()` for app URLs**: Always use `include('app.urls')` in the project's `urls.py` rather than defining all URLs in one file.
- **Name your URL patterns**: Always add `name='...'` to `path()` calls so you can reference URLs by name in templates and views.

## Summary

- A Django **project** contains configuration; an **app** contains functionality
- Create an app with `python manage.py startapp appname`
- Register the app in `INSTALLED_APPS` in `settings.py`
- Each app has its own `models.py`, `views.py`, `admin.py`, and `tests.py`
- Use `include()` to wire app URLs into the project's URL configuration

## Code Examples

**Creating a basic view and URL configuration for a Django app**

```python
# polls/views.py
from django.http import HttpResponse

def index(request):
    return HttpResponse("Hello, world! This is the polls index.")

# polls/urls.py
from django.urls import path
from . import views

urlpatterns = [
    path('', views.index, name='index'),
]
```


## Resources

- [Django Applications](https://docs.djangoproject.com/en/6.0/ref/applications/) — Official documentation on Django applications

---

> 📘 *This lesson is part of the [Django Foundations](https://stanza.dev/courses/django-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*