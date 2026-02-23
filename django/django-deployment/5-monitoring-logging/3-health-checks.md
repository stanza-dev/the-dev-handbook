---
source_course: "django-deployment"
source_lesson: "django-deployment-health-checks"
---

# Health Check Endpoints

## Introduction

Health check endpoints allow load balancers and orchestrators like Kubernetes to verify your Django application is functioning correctly. A proper health check verifies not just that the process is running, but that database and cache connections are healthy.

## Types of Checks

**Liveness**: Is the application running?

**Readiness**: Is the application ready for traffic?

**Deep Check**: Are dependencies (DB, cache) working?

## Implementation

```python
from django.http import JsonResponse
from django.db import connection

def health(request):
    return JsonResponse({'status': 'ok'})

def health_deep(request):
    checks = {}
    healthy = True
    
    try:
        connection.ensure_connection()
        checks['database'] = 'ok'
    except:
        checks['database'] = 'failed'
        healthy = False
    
    status = 200 if healthy else 503
    return JsonResponse({'status': 'healthy' if healthy else 'unhealthy', 'checks': checks}, status=status)
```

## Kubernetes Probes

```yaml
livenessProbe:
  httpGet:
    path: /health/
    port: 8000
readinessProbe:
  httpGet:
    path: /health/ready/
    port: 8000
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

1. **Separate liveness/readiness**: Different purposes.
2. **Check dependencies**: Database, cache, queues.
3. **Return proper status codes**: 200 or 503.

## Summary

Health checks let load balancers and orchestrators know if your app is working. Return 200 for healthy, 503 for unhealthy. Check database and cache connectivity.

## Code Examples

**Health check endpoint verifying database and cache connectivity**

```yaml
from django.http import JsonResponse
from django.db import connection

def health(request):
    return JsonResponse({'status': 'ok'})

def health_deep(request):
    checks = {}
    healthy = True
    
    try:
        connection.ensure_connection()
        checks['database'] = 'ok'
    except:
        checks['database'] = 'failed'
        healthy = False
    
    status = 200 if healthy else 503
    return JsonResponse({'status': 'healthy' if healthy else 'unhealthy', 'checks': checks}, status=status)
```


## Resources

- [Kubernetes Probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/) — Kubernetes health probes

---

> 📘 *This lesson is part of the [Django Deployment & Production](https://stanza.dev/courses/django-deployment) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*