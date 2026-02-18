---
source_course: "django-deployment"
source_lesson: "django-deployment-health-checks"
---

# Health Check Endpoints

## Introduction

Health checks verify your application is working and ready to receive traffic.

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

## Best Practices

1. **Separate liveness/readiness**: Different purposes.
2. **Check dependencies**: Database, cache, queues.
3. **Return proper status codes**: 200 or 503.

## Summary

Health checks let load balancers and orchestrators know if your app is working. Return 200 for healthy, 503 for unhealthy. Check database and cache connectivity.

## Resources

- [Kubernetes Probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/) — Kubernetes health probes

---

> 📘 *This lesson is part of the [Django Deployment & Production](https://stanza.dev/courses/django-deployment) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*