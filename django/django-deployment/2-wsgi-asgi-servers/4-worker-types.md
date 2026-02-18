---
source_course: "django-deployment"
source_lesson: "django-deployment-worker-types"
---

# Worker Types and Configuration

## Introduction

Different worker types suit different workloads. Understanding when to use sync, async, or threaded workers optimizes performance.

## Worker Types

**Sync (default)**: One request per worker. Good for CPU-bound tasks.

**Gevent**: Async I/O. Good for I/O-bound tasks, many connections.

**Threads**: Multiple threads per worker. Balanced approach.

## Configuration

```python
# gunicorn.conf.py
import multiprocessing

# Sync (CPU-bound)
worker_class = 'sync'
workers = (2 * multiprocessing.cpu_count()) + 1

# Gevent (I/O-bound)
# pip install gevent
worker_class = 'gevent'
worker_connections = 1000
workers = multiprocessing.cpu_count()

# Threads (balanced)
worker_class = 'gthread'
workers = multiprocessing.cpu_count()
threads = 4
```

## Best Practices

1. **Start with sync**: Simplest, adjust based on monitoring.
2. **Use gevent for APIs**: Better for database/API heavy apps.
3. **Monitor and tune**: Adjust based on actual performance.

## Summary

Choose worker type based on workload: sync for CPU-bound, gevent for I/O-bound, threads for mixed. Start with (2 × CPU) + 1 workers and tune from there.

## Resources

- [Gunicorn Design](https://docs.gunicorn.org/en/stable/design.html) — Gunicorn worker design

---

> 📘 *This lesson is part of the [Django Deployment & Production](https://stanza.dev/courses/django-deployment) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*