---
source_course: "django-rest-api"
source_lesson: "django-rest-api-performance-testing"
---

# Performance Testing APIs

## Introduction

APIs that work in tests may fail under production load. Performance testing identifies bottlenecks before they impact users.

## Key Concepts

**Load Testing**: Testing API behavior under expected traffic levels.

**Stress Testing**: Pushing beyond normal limits to find breaking points.

**N+1 Queries**: Database inefficiency where N additional queries are made for N results.

## Real World Context

Performance testing prevents production outages:
- **Black Friday traffic**: E-commerce APIs must handle 10-100x normal load
- **API launches**: New integrations can spike traffic unexpectedly
- **Data growth**: APIs fast with 1,000 records may crawl with 1,000,000
- **N+1 queries**: A single endpoint can generate hundreds of database queries without proper optimization

## Deep Dive

### Detecting N+1 Queries

This test creates 10 articles and then asserts that listing them requires fewer than 5 database queries, catching N+1 problems automatically:

```python
from django.test import TestCase
from django.test.utils import override_settings
from django.db import connection, reset_queries

class QueryCountTests(TestCase):
    @override_settings(DEBUG=True)
    def test_list_articles_query_count(self):
        # Create test data
        for _ in range(10):
            ArticleFactory()
        
        reset_queries()
        
        response = self.client.get('/api/articles/')
        
        # Should be 1-2 queries, not 11+
        query_count = len(connection.queries)
        self.assertLess(query_count, 5, f'Too many queries: {query_count}')
```

The `@override_settings(DEBUG=True)` decorator enables query logging, and `reset_queries()` clears the log before the request under test. If someone removes `select_related()`, this test catches it.

### Using django-silk for Profiling

django-silk records every request's SQL queries, execution time, and profiling data in a browsable dashboard:

```python
# pip install django-silk

# settings.py
INSTALLED_APPS = ['silk']
MIDDLEWARE = ['silk.middleware.SilkyMiddleware']

# Access profiling at /silk/
```

Once installed, visit `/silk/` in your browser to see query counts, response times, and detailed SQL logs for each request.

### Load Testing with locust

Locust lets you define user behavior as Python code and simulate hundreds of concurrent users hitting your API:

```python
# locustfile.py
from locust import HttpUser, task, between

class APIUser(HttpUser):
    wait_time = between(1, 3)
    
    @task(3)
    def list_articles(self):
        self.client.get('/api/articles/')
    
    @task(1)
    def get_article(self):
        self.client.get('/api/articles/1/')
    
    @task(1)
    def create_article(self):
        self.client.post('/api/articles/',
            json={'title': 'Test', 'body': 'Content'},
            headers={'Authorization': f'Bearer {self.token}'}
        )

# Run: locust -f locustfile.py --host=http://localhost:8000
```

The `@task` decorator's weight parameter controls how often each action runs. Here, listing articles runs 3x more often than creating one, simulating realistic read-heavy traffic.

## Common Pitfalls

1. **Testing with small datasets**: An endpoint fast with 10 records may be slow with 10,000. Test with realistic data volumes.

2. **Ignoring query counts**: An endpoint making 100 queries instead of 2 will cause problems at scale.

3. **Not profiling before optimizing**: Guessing at bottlenecks wastes time. Use Django Debug Toolbar or django-silk to find actual issues.

## Best Practices

1. **Count queries in tests**: Fail tests that exceed expected query counts.
2. **Profile before optimizing**: Find real bottlenecks with profiling tools.
3. **Test with realistic data**: Small test datasets may hide performance issues.

## Summary

Performance testing prevents production outages. Count database queries in unit tests, use profiling tools to identify bottlenecks, and run load tests to validate capacity.

## Code Examples

**Detecting N+1 queries by counting database queries in tests**

```python
from django.test import TestCase
from django.test.utils import override_settings
from django.db import connection, reset_queries

class QueryCountTests(TestCase):
    @override_settings(DEBUG=True)
    def test_list_articles_query_count(self):
        for _ in range(10):
            ArticleFactory()
        reset_queries()
        response = self.client.get('/api/articles/')
        query_count = len(connection.queries)
        self.assertLess(query_count, 5, f'Too many queries: {query_count}')
```


## Resources

- [Django Database Optimization](https://docs.djangoproject.com/en/6.0/topics/db/optimization/) — Database access optimization

---

> 📘 *This lesson is part of the [Django REST API Development](https://stanza.dev/courses/django-rest-api) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*