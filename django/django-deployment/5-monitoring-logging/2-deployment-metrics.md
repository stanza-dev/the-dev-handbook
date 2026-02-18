---
source_course: "django-deployment"
source_lesson: "django-deployment-metrics"
---

# Application Metrics

## Introduction

Metrics provide quantitative data about your application's performance and health.

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

## Best Practices

1. **Track RED metrics**: Rate, Errors, Duration.
2. **Use labels wisely**: Avoid high cardinality.
3. **Set up dashboards**: Grafana for visualization.

## Summary

Metrics help you understand application performance. Track request rate, latency, and errors. Use Prometheus and Grafana for collection and visualization.

## Resources

- [django-prometheus](https://github.com/korfuri/django-prometheus) — Django Prometheus integration

---

> 📘 *This lesson is part of the [Django Deployment & Production](https://stanza.dev/courses/django-deployment) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*