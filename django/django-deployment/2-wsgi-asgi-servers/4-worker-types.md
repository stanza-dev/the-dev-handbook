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

1. **Start with sync**: Simplest, adjust based on monitoring.
2. **Use gevent for APIs**: Better for database/API heavy apps.
3. **Monitor and tune**: Adjust based on actual performance.

## Summary

Choose worker type based on workload: sync for CPU-bound, gevent for I/O-bound, threads for mixed. Start with (2 × CPU) + 1 workers and tune from there.

## Code Examples

**Gunicorn worker type configurations — sync, gevent, and threaded**

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


## Resources

- [Gunicorn Design](https://docs.gunicorn.org/en/stable/design.html) — Gunicorn worker design

---

> 📘 *This lesson is part of the [Django Deployment & Production](https://stanza.dev/courses/django-deployment) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*