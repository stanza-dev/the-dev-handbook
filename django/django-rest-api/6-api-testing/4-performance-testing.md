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

## Deep Dive

### Detecting N+1 Queries

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

### Using django-silk for Profiling

```python
# pip install django-silk

# settings.py
INSTALLED_APPS = ['silk']
MIDDLEWARE = ['silk.middleware.SilkyMiddleware']

# Access profiling at /silk/
```

### Load Testing with locust

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

## Best Practices

1. **Count queries in tests**: Fail tests that exceed expected query counts.
2. **Profile before optimizing**: Find real bottlenecks with profiling tools.
3. **Test with realistic data**: Small test datasets may hide performance issues.

## Summary

Performance testing prevents production outages. Count database queries in unit tests, use profiling tools to identify bottlenecks, and run load tests to validate capacity.

## Resources

- [Django Database Optimization](https://docs.djangoproject.com/en/6.0/topics/db/optimization/) — Database access optimization

---

> 📘 *This lesson is part of the [Django REST API Development](https://stanza.dev/courses/django-rest-api) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*