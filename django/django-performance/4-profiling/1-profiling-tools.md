---
source_course: "django-performance"
source_lesson: "django-performance-profiling-tools"
---

# Performance Profiling

Profiling helps identify where your application spends its time.

## Django Debug Toolbar

```python
# pip install django-debug-toolbar

# settings.py
INSTALLED_APPS = [
    # ...
    'debug_toolbar',
]

MIDDLEWARE = [
    'debug_toolbar.middleware.DebugToolbarMiddleware',
    # ...
]

INTERNAL_IPS = ['127.0.0.1']

# urls.py
if settings.DEBUG:
    import debug_toolbar
    urlpatterns = [
        path('__debug__/', include(debug_toolbar.urls)),
    ] + urlpatterns
```

## Query Logging

```python
# settings.py - Log all SQL queries
LOGGING = {
    'version': 1,
    'handlers': {
        'console': {
            'class': 'logging.StreamHandler',
        },
    },
    'loggers': {
        'django.db.backends': {
            'level': 'DEBUG',
            'handlers': ['console'],
        },
    },
}
```

## Programmatic Query Counting

```python
from django.db import connection, reset_queries
from django.conf import settings


def count_queries(func):
    """Decorator to count queries."""
    def wrapper(*args, **kwargs):
        reset_queries()
        result = func(*args, **kwargs)
        print(f'{func.__name__}: {len(connection.queries)} queries')
        return result
    return wrapper


# Context manager
class QueryCounter:
    def __enter__(self):
        reset_queries()
        return self
    
    def __exit__(self, *args):
        self.count = len(connection.queries)
        print(f'Queries executed: {self.count}')
        for query in connection.queries:
            print(f"  {query['time']}s: {query['sql'][:100]}")


# Usage
with QueryCounter():
    articles = list(Article.objects.select_related('author').all())
```

## cProfile

```python
import cProfile
import pstats
import io


def profile_view(view_func):
    """Decorator to profile a view."""
    def wrapper(request, *args, **kwargs):
        profiler = cProfile.Profile()
        profiler.enable()
        
        response = view_func(request, *args, **kwargs)
        
        profiler.disable()
        
        # Output stats
        stream = io.StringIO()
        stats = pstats.Stats(profiler, stream=stream)
        stats.sort_stats('cumulative')
        stats.print_stats(20)  # Top 20 functions
        print(stream.getvalue())
        
        return response
    return wrapper


@profile_view
def slow_view(request):
    # Your view code
    pass
```

## Silk Profiler

```python
# pip install django-silk

# settings.py
INSTALLED_APPS = [
    # ...
    'silk',
]

MIDDLEWARE = [
    # ...
    'silk.middleware.SilkyMiddleware',
]

# urls.py
urlpatterns += [path('silk/', include('silk.urls'))]

# Run migrations
# python manage.py migrate

# Profile specific functions
from silk.profiling.profiler import silk_profile

@silk_profile(name='Complex Calculation')
def complex_calculation():
    # Your code
    pass
```

## Memory Profiling

```python
# pip install memory_profiler

from memory_profiler import profile


@profile
def memory_heavy_function():
    data = []
    for i in range(1000000):
        data.append(i * i)
    return data
```

## Locust Load Testing

```python
# pip install locust
# locustfile.py

from locust import HttpUser, task, between


class WebsiteUser(HttpUser):
    wait_time = between(1, 5)
    
    @task(3)
    def view_articles(self):
        self.client.get('/articles/')
    
    @task(1)
    def view_article(self):
        self.client.get('/articles/sample-article/')
    
    @task(2)
    def search(self):
        self.client.get('/search/?q=django')

# Run: locust -f locustfile.py --host=http://localhost:8000
```

## Resources

- [Django Debug Toolbar](https://django-debug-toolbar.readthedocs.io/) — Debug toolbar documentation

---

> 📘 *This lesson is part of the [Django Performance & Optimization](https://stanza.dev/courses/django-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*