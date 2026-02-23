---
source_course: "django-deployment"
source_lesson: "django-deployment-uvicorn-asgi"
---

# Uvicorn ASGI Server

## Introduction

Uvicorn is a high-performance ASGI server that enables Django's async features including async views, WebSockets, and concurrent I/O. For production deployments, it's typically paired with Gunicorn as a process manager using Uvicorn workers.

## Key Concepts

- **ASGI**: Asynchronous Server Gateway Interface — the async evolution of WSGI that supports WebSockets and long-lived connections.
- **Uvicorn**: A fast ASGI server built on uvloop and httptools for maximum performance.
- **Daphne**: An alternative ASGI server from the Django Channels project, particularly strong for WebSocket support.
- **UvicornWorker**: A Gunicorn worker class that runs Uvicorn's event loop, combining Gunicorn's process management with Uvicorn's async capabilities.

## Real World Context

Teams adopt ASGI when they need real-time features like WebSocket chat, live notifications, or server-sent events. The Gunicorn + UvicornWorker pattern is the recommended production setup because Gunicorn handles process management (spawning workers, graceful restarts) while Uvicorn handles async request processing. This is simpler to manage than running multiple standalone Uvicorn processes.

## Deep Dive

For Django applications using async views or WebSockets, Uvicorn provides an ASGI server that can be run standalone or as Gunicorn workers for process management.


## Introduction

Uvicorn is a lightning-fast ASGI server for async Django.

## Basic Setup

```bash
# Install
pip install uvicorn

# Run
uvicorn myproject.asgi:application

# With options
uvicorn myproject.asgi:application \
    --host 0.0.0.0 \
    --port 8000 \
    --workers 4
```

## ASGI Configuration

```python
# myproject/asgi.py
import os
from django.core.asgi import get_asgi_application

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'myproject.settings.production')

application = get_asgi_application()
```

## Production Configuration

```bash
# Production command
uvicorn myproject.asgi:application \
    --host 0.0.0.0 \
    --port 8000 \
    --workers 4 \
    --loop uvloop \
    --http httptools \
    --log-level info \
    --access-log
```

## Gunicorn with Uvicorn Workers

```bash
# Best of both worlds
gunicorn myproject.asgi:application \
    -w 4 \
    -k uvicorn.workers.UvicornWorker \
    --bind 0.0.0.0:8000
```

```python
# gunicorn.conf.py
workers = 4
worker_class = 'uvicorn.workers.UvicornWorker'
bind = '0.0.0.0:8000'
```

## Daphne ASGI Server

```bash
# Alternative ASGI server (good for WebSockets)
pip install daphne

# Run
daphne -b 0.0.0.0 -p 8000 myproject.asgi:application

# With multiple workers
daphne -b 0.0.0.0 -p 8000 \
    --proxy-headers \
    myproject.asgi:application
```

## Systemd for Uvicorn

```ini
# /etc/systemd/system/uvicorn.service
[Unit]
Description=Uvicorn ASGI server for myproject
After=network.target

[Service]
User=www-data
Group=www-data
WorkingDirectory=/var/www/myproject
EnvironmentFile=/var/www/myproject/.env
ExecStart=/var/www/myproject/venv/bin/uvicorn \
    myproject.asgi:application \
    --host 0.0.0.0 \
    --port 8000 \
    --workers 4 \
    --loop uvloop
Restart=always
RestartSec=3

[Install]
WantedBy=multi-user.target
```

## When to Use ASGI vs WSGI

```python
# Use WSGI (Gunicorn) when:
# - Your app is mostly synchronous
# - You don't need WebSockets
# - Maximum compatibility is needed

# Use ASGI (Uvicorn/Daphne) when:
# - You have async views
# - You need WebSockets
# - You're using Django Channels
# - You want better concurrency for I/O-bound operations
```

## Summary

- The techniques covered in this lesson are essential for production-quality applications.
- Always measure before and after optimizing to verify improvements.
- Start with the simplest approach and add complexity only when needed.

## Common Pitfalls

1. **Using ASGI when you don't need async** — If your application is entirely synchronous, ASGI adds complexity without benefit. Stick with WSGI/Gunicorn.
2. **Not installing uvloop** — Uvicorn performs significantly better with uvloop, but it's an optional dependency you must install explicitly.
3. **Using CONN_MAX_AGE with ASGI** — Django docs recommend keeping CONN_MAX_AGE at 0 under ASGI and using an external connection pooler like PgBouncer instead.

## Best Practices

1. **Use Gunicorn + UvicornWorker for production** — This gives you process management, graceful restarts, and async support in one command.
2. **Install uvloop and httptools** — These C-based libraries dramatically improve Uvicorn's throughput.
3. **Test thoroughly under ASGI** — Some Django middleware and third-party packages may not be fully async-compatible. Test your full middleware stack.

## Summary

- The techniques covered in this lesson are essential for production-quality Django applications.
- Always measure and profile before optimizing to ensure you're addressing the actual bottleneck.
- Start with the simplest approach and add complexity only when monitoring shows it's needed.

## Code Examples

**Running Django with Gunicorn and Uvicorn workers for ASGI support**

```python
# Use WSGI (Gunicorn) when:
# - Your app is mostly synchronous
# - You don't need WebSockets
# - Maximum compatibility is needed

# Use ASGI (Uvicorn/Daphne) when:
# - You have async views
# - You need WebSockets
# - You're using Django Channels
# - You want better concurrency for I/O-bound operations
```


## Resources

- [ASGI Servers](https://docs.djangoproject.com/en/6.0/howto/deployment/asgi/) — Django ASGI deployment

---

> 📘 *This lesson is part of the [Django Deployment & Production](https://stanza.dev/courses/django-deployment) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*