---
source_course: "django-performance"
source_lesson: "django-performance-load-testing"
---

# Load Testing with Locust

## Introduction

Load testing reveals performance issues before they affect users. Locust makes it easy to write Python-based load tests.

## Key Concepts

- **Load Testing**: Simulating concurrent users to measure how an application performs under stress.
- **Locust**: A Python-based load testing tool where user behavior is defined in code, making it easy to version-control and integrate with CI.
- **Throughput**: Requests per second (RPS) the application can handle at a given concurrency level.
- **Percentile Latency**: Response time at a given percentile (e.g., p95 = 95% of requests are faster than this). More meaningful than average latency.

## Real World Context

This topic directly impacts production application performance. Teams that master these techniques reduce page load times, lower infrastructure costs, and deliver better user experiences.

## Deep Dive

### Basic Locust File

```python
# locustfile.py
from locust import HttpUser, task, between

class WebsiteUser(HttpUser):
    wait_time = between(1, 5)  # Wait 1-5s between tasks
    
    @task(3)  # Weight: 3x more likely
    def view_articles(self):
        self.client.get('/articles/')
    
    @task(1)
    def view_article(self):
        self.client.get('/articles/sample-article/')
    
    @task(2)
    def search(self):
        self.client.get('/search/', params={'q': 'django'})
    
    def on_start(self):
        # Login once at start
        self.client.post('/login/', {
            'username': 'testuser',
            'password': 'testpass'
        })
```

### Running Locust

```bash
# Start with web UI
locust -f locustfile.py --host=http://localhost:8000

# Headless mode for CI
locust -f locustfile.py --headless \
    --users 100 \
    --spawn-rate 10 \
    --run-time 1m \
    --host=http://localhost:8000
```

### Testing Authenticated Endpoints

```python
class AuthenticatedUser(HttpUser):
    token = None
    
    def on_start(self):
        response = self.client.post('/api/token/', {
            'username': 'test',
            'password': 'pass'
        })
        self.token = response.json()['access']
    
    @task
    def get_profile(self):
        self.client.get('/api/profile/', headers={
            'Authorization': f'Bearer {self.token}'
        })
```

## Common Pitfalls

1. **Premature optimization** — Always profile before optimizing. Fix the biggest bottleneck first rather than guessing.
2. **Ignoring trade-offs** — Every optimization has costs. Caching adds complexity, indexes slow writes, and async adds cognitive overhead.

## Best Practices

1. **Test against staging, not production**: Use production-like data.
2. **Start small**: Gradually increase load to find breaking points.
3. **Monitor during tests**: Watch server metrics alongside Locust.

## Summary

Use Locust to simulate real user load. Define realistic user scenarios with weights. Monitor server metrics during tests to find bottlenecks.

## Code Examples

**Locust load test file simulating weighted user scenarios**

```python
# locustfile.py
from locust import HttpUser, task, between

class WebsiteUser(HttpUser):
    wait_time = between(1, 5)  # Wait 1-5s between tasks
    
    @task(3)  # Weight: 3x more likely
    def view_articles(self):
        self.client.get('/articles/')
    
    @task(1)
    def view_article(self):
        self.client.get('/articles/sample-article/')
    
    @task(2)
    def search(self):
        self.client.get('/search/', params={'q': 'django'})
    
    def on_start(self):
        # Login once at start
# ...
```


## Resources

- [Locust](https://docs.locust.io/) — Locust load testing documentation

---

> 📘 *This lesson is part of the [Django Performance & Optimization](https://stanza.dev/courses/django-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*