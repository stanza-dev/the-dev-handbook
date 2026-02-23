---
source_course: "django-performance"
source_lesson: "django-performance-profiling-tools"
---

# Performance Profiling

## Introduction

Profiling is the systematic measurement of where your application spends its time and memory. Before optimizing anything, you must profile to identify the actual bottleneck — otherwise you risk spending effort on code that isn't the real problem.

## Key Concepts

- **Django Debug Toolbar**: A browser-based panel showing SQL queries, template rendering time, cache operations, and more during development.
- **cProfile**: Python's built-in deterministic profiler that measures function call counts and cumulative time.
- **EXPLAIN**: A SQL command that shows the database query execution plan, revealing whether indexes are used.
- **Load Testing**: Simulating multiple concurrent users to find performance bottlenecks under realistic traffic.

## Real World Context

Every production Django application benefits from profiling. Django Debug Toolbar is typically the first tool installed on new projects — it immediately reveals N+1 queries, slow template renders, and excessive cache misses. For production profiling, tools like Silk and Sentry Performance show real user latency data. Load testing with Locust before launches prevents embarrassing failures under traffic spikes.

## Deep Dive

Effective profiling requires measuring real query execution, not guessing. Django provides several tools from basic query logging to detailed execution plan analysis.


## Introduction

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

## Summary

- The techniques covered in this lesson are essential for production-quality applications.
- Always measure before and after optimizing to verify improvements.
- Start with the simplest approach and add complexity only when needed.

## Common Pitfalls

1. **Profiling in production with DEBUG=True** — Query logging under DEBUG=True adds significant overhead and stores all queries in memory, which can cause out-of-memory errors under load.
2. **Optimizing without profiling first** — Developer intuition about bottlenecks is often wrong. Always measure before optimizing.
3. **Only testing with small datasets** — A query that runs in 1ms with 100 rows can take 10 seconds with 1 million rows. Profile with production-sized data.

## Best Practices

1. **Install Django Debug Toolbar on day one** — It catches performance issues during development before they reach production.
2. **Use `assertNumQueries` in tests** — Django's test assertion catches N+1 regressions automatically in your CI pipeline.
3. **Profile the critical path** — Focus on the pages and API endpoints that users hit most frequently.

## Summary

- The techniques covered in this lesson are essential for production-quality Django applications.
- Always measure and profile before optimizing to ensure you're addressing the actual bottleneck.
- Start with the simplest approach and add complexity only when monitoring shows it's needed.

## Code Examples

**Query counting context manager for identifying N+1 queries**

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
    
# ...
```


## Resources

- [Django Debug Toolbar](https://django-debug-toolbar.readthedocs.io/) — Debug toolbar documentation

---

> 📘 *This lesson is part of the [Django Performance & Optimization](https://stanza.dev/courses/django-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*