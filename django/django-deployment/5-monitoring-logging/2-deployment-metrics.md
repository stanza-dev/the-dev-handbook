---
source_course: "django-deployment"
source_lesson: "django-deployment-metrics"
---

# Application Metrics

## Introduction

Application metrics provide quantitative insight into how your Django application performs under real load. Prometheus-style metrics track request rates, error percentages, and latency distributions — the RED method (Rate, Errors, Duration) that powers modern observability.

## Key Metrics

**Request Rate**: Requests per second.

**Latency**: Response time (p50, p95, p99).

**Error Rate**: Percentage of failed requests.

**Saturation**: Resource utilization (CPU, memory).

## django-prometheus Setup

```python
# pip install django-prometheus

INSTALLED_APPS = ['django_prometheus']

MIDDLEWARE = [
    'django_prometheus.middleware.PrometheusBeforeMiddleware',
    ...,
    'django_prometheus.middleware.PrometheusAfterMiddleware',
]

# urls.py
path('metrics/', include('django_prometheus.urls')),
```

## Custom Metrics

```python
from prometheus_client import Counter, Histogram

ORDERS = Counter('orders_total', 'Total orders', ['status'])
LATENCY = Histogram('api_latency_seconds', 'API latency')

@LATENCY.time()
def api_view(request):
    ORDERS.labels(status='success').inc()
```

## Key Concepts

The key terms and concepts for this topic are introduced in the Deep Dive section below.


## Deep Dive

See the detailed technical content and code examples throughout this lesson.

## Real World Context

This topic directly impacts production application performance. Teams that master these techniques reduce page load times, lower infrastructure costs, and deliver better user experiences.

## Common Pitfalls

1. **Premature optimization** — Always profile before optimizing. Fix the biggest bottleneck first rather than guessing.
2. **Ignoring trade-offs** — Every optimization has costs. Caching adds complexity, indexes slow writes, and async adds cognitive overhead.

## Best Practices

1. **Track RED metrics**: Rate, Errors, Duration.
2. **Use labels wisely**: Avoid high cardinality.
3. **Set up dashboards**: Grafana for visualization.

## Summary

Metrics help you understand application performance. Track request rate, latency, and errors. Use Prometheus and Grafana for collection and visualization.

## Code Examples

**Custom Prometheus metrics with Counter and Histogram in Django**

```python
# pip install django-prometheus

INSTALLED_APPS = ['django_prometheus']

MIDDLEWARE = [
    'django_prometheus.middleware.PrometheusBeforeMiddleware',
    ...,
    'django_prometheus.middleware.PrometheusAfterMiddleware',
]

# urls.py
path('metrics/', include('django_prometheus.urls')),
```


## Resources

- [django-prometheus](https://github.com/korfuri/django-prometheus) — Django Prometheus integration

---

> 📘 *This lesson is part of the [Django Deployment & Production](https://stanza.dev/courses/django-deployment) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*